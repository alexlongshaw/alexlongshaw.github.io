---
layout: post
title:  "Setting up DNS for ELB Route 53 across multiple accounts"
date:   2016-07-16 20:22:00 +0000
categories: aws
disqus_id: b6bf5e5a-e980-474f-b1b0-767117954435
---

Our testing environments are setup in the configuration “client-site.testing.our-company.com”, using AWS Route 53 to manage DNS records. This is hosted on our client’s AWS account, however our DNS is configured through Route 53 on our own account.

Previously, we have just used a CNAME to the loadbalancer, which worked fine for our needs until [Mandrill required domain and subdomain verification](https://mandrill.zendesk.com/hc/en-us/articles/205582247).

To verify, we had to have an email sent to alex@client-site.testing.our-company.com, despite already verifying our-company.com. This meant I had to setup MX records at client-site.testing.our-company.com, which can’t be added when there is a CNAME with the same value.

The official recommendation is to use an A record, documented under the heading “[To create a hosted zone and an alias record set for your domain using Amazon Route 53](http://docs.aws.amazon.com/ElasticLoadBalancing/latest/DeveloperGuide/using-domain-names-with-elb.html)”.

What I couldn’t find documented is that you can do this across accounts. You won’t get the auto completion you get if your load balancer is on the same account but it works in the same way.
