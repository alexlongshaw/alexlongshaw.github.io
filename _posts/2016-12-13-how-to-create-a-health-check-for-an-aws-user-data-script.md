---
layout: post
title:  "How to create a health check for an AWS user data script"
date:   2016-12-13 12:30:34 +0000
categories: aws
---

### Introduction

We use user data scripts when setting up servers. (I’m not sure why they’re called that, it’s actually just a script that runs when a new instance is created). We had some unexpected errors on production and found that certain services were not running which were set up in the user data script due to a failure at the start of the script. This caused one of our schedulers running on a cron to break.

### The 5 whys

After the error occurred, we terminated the instance manually which restarted automatically as it uses auto scaling groups, therefore running the user data script again. However, we wanted to look deeper into what had caused it and what we could do to prevent it from happening again.

Kirsten had read about a strategy called the 5 whys in The Lean Startup by Eric Ries and suggested we try it out as it’s a good technique for digging down into the root cause which helps to not just fix problems on a shallow level.

In order to fix the problem, we needed to check the status of the user data script and ideally, automatically fix the instance. We wanted visibility of the problem (logs/alarms) and ideally for it to self heal and so decided to allow 5 minutes for an instance to have not completed its user data script before forcing the server to be destroyed and run the process again.

For the purpose of this post, I’ve used an Ubuntu server, managed with Packer and Terraform, but everything in this guide can be done manually.

### Improving visibility

To check if our user data script has finished running correctly, we’re going to create a health check and run it every minute for 5 minutes. In order to know that the user data script has completed successfully, we’ll need to create a complete flag we can check against later on. Our updated user data script is below:

```
#!/bin/sh

export HOME=/home/ubuntu
export AWS_CONFIG_FILE=/home/ubuntu/.aws/config

# Define the tag you want to get the value for
KEY=Environment
# Grab instance ID and region as the 'describe-tags' action below requires them. Getting the region
INSTANCE_ID=$(/usr/bin/ec2metadata --instance-id)
REGION=$(/usr/bin/curl -s http://169.254.169.254/latest/dynamic/instance-identity/document | grep region | awk -F\" '{print $4}')

# Grab tag value
TAG_VALUE=$(/usr/local/bin/aws ec2 describe-tags --filters "Name=resource-id,Values=$INSTANCE_ID" "Name=key,Values=$KEY" --region=$REGION --output=text | cut -f5)

# ... environment and services setup

echo "
[worker_syslog]
file = /var/log/syslog
log_group_name = $TAG_VALUE
log_stream_name = worker_syslog
" >> /var/awslogs/etc/awslogs.conf

# ... final setup

# Restart AWS logs
service awslogs restart

# Create user data success flag
touch /home/ubuntu/health/complete
```

Now we need to create our health check script:

```
#!/bin/sh

COMPLETE_FILE=/home/ubuntu/health/complete

# There is no complete file, investigate further
if [ ! -f "$COMPLETE_FILE" ]; then

    LOG_FILE=/home/ubuntu/health/counter.log;

    if [ ! -f $LOG_FILE ]; then
        echo 1 > $LOG_FILE
    else
        existingCount=$(head -n 1 $LOG_FILE)
        currentCount=$(($existingCount + 1))
        echo $currentCount > $LOG_FILE
    fi
fi
```

This will check if a complete file has been created, and if not increment a counter each time it’s run. In order to run the file, you’ll need to make it executable:

`chmod +x health-check.sh`

For this script to have any use, it needs to be run automatically. The following cron will run the script every minute:

`* * * * * /home/ubuntu/health/health-check.sh`

On Ubuntu, cron jobs can be set up by adding a new line to your crontab file by using the “crontab -e” command.

As we’re not sitting at our desks with logs streaming away in front of us, we need an alert to tell us when there’s an issue. To do this, we’re going to use a CloudWatch Alarm which can be configured to send an email to the development team when a line in the logs matches a pattern. When matching anything other than text, numbers or underscores, make sure you wrap what you want to match in quotes or it won’t return the results you expect.

We set this up in Terraform but it can be done manually in CloudWatch. First we need a metric. To create this, select the logs you want your metrics for and press the ‘Create Metric Filter’ button. Add the filter pattern to match a section of your logs which in my case was “SYSLOG-ERROR”. Test your pattern to make sure that it matches what you expect and click Assign Metric. Fill in the metric details and create your filter.

You can directly create an alarm from the confirmation screen. We just need to give it a name, description, select the notification list and set the threshold to greater or equal than 1 for one consecutive period, and set the period to 1 minute. Press the Create Alarm button and you’re done. You should now have a functioning alarm for when your user data script fails.

### Self healing

Now we have our monitoring in place, we want it to take corrective action itself. As we mentioned earlier, our fix for this was simply to terminate the unhealthy instance and let the auto scaling group recreate the instance, and hence run the user data script again.

```
#!/bin/sh

COMPLETE_FILE=/home/ubuntu/health/complete

# There is no complete file, investigate further
if [ ! -f "$COMPLETE_FILE" ]; then

    LOG_FILE=/home/ubuntu/health/counter.log;

    if [ ! -f $LOG_FILE ]; then
        echo 1 > $LOG_FILE
    else
        existingCount=$(head -n 1 $LOG_FILE)
        currentCount=$(($existingCount + 1))
        echo $currentCount > $LOG_FILE

        # Complete file has been missing for 5 minutes, destroy the instance
        if [ "$currentCount" -gt "5" ]; then
            INSTANCE_ID=$(/usr/bin/ec2metadata --instance-id)

            logger -p local0.notice -t $0[$$] "SYSLOG-ERROR: User data script failed. Setting instance $INSTANCE_ID as Unhealthy. Complete file should be in $COMPLETE_FILE but was not found after 5 minutes."

            /usr/local/bin/aws autoscaling set-instance-health --instance-id "$INSTANCE_ID" --health-status Unhealthy
        fi
    fi
fi
```

### Going forward

We could improve this by checking if the log has appeared and only triggering an alarm if it goes off multiple times. This health check has been in production for several months and is working well so we may make this enhancement when it’s more mature.

*This was originally posted to UVD, but is no longer available in its original source.*