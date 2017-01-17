---
layout: post
title:  "How to create a health check for an AWS user data script"
date:   2016-12-13 12:30:34 +0000
link:   https://www.uvd.co.uk/blog/create-health-check-aws-user-data-script/
categories: aws
disqus_id: 586c67aa-dd09-11e6-bf26-cec0c932ce01
---

After one of our user data scripts caused one of our schedulers running on a cron to break, we decided to set up an alarm to tell us when it was happening and also set up a fix to heal itself. This post outlines the steps we took.
