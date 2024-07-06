---
layout: post
title:  "How to set up AWS Certificate Manager with NGINX"
date:   2016-05-27 14:30:00 +0000
categories: aws
---

### How we chose AWS certificate manager

I was recently tasked with getting an SSL certificate for uvd.co.uk (or TLS I should say). We’ve used them on a few websites before which has worked great in the production environment, but we’ve always had ugly ‘insecure content’ warnings on our testing and staging environments. I was keen to avoid this and not wanting to buy two certificates, I looked at the free offerings.

The main player that’s had a lot of press attention is [Let’s Encrypt](https://letsencrypt.org/). They’ve been doing some great work and whilst being used in production environments, it still takes some work to setup and requires renewal every 90 days. This can be automated but caused me to look at what else was out there.

Earlier this year, Amazon released [AWS Certificate Manager (ACM)](https://aws.amazon.com/certificate-manager/), which provides free SSL certificates for its customers. All you have to do it pay for the resources. You can only use these on Elastic Load Balancing (ELB), which is great for us as that’s exactly what we use.

### So, let’s get started.

I’m going to assume you currently have a project, served by NGINX behind a load balancer.

From the AWS dashboard, go to “Certificate Manager” under “Security & Identity”. Before you request a certificate, make sure you have access to one of the following email accounts:

- administrator@yourdomain.com
- postmaster@yourdomain.com
- webmaster@yourdomain.com
- hostmaster@yourdomain.com
- admin@yourdomain.com

Now to request a certificate. I registered uvd.co.uk and *.uvd.co.uk as I have some subdomains that I want to use under the load balancer.

The certificate will now be in the “Pending validation” state. Check the emails of one of the accounts listed above and approve the verification. It can take a couple of minutes to approve.

When the status is “Issued” we can add it to the load balancer. Select your load balancer and go to the “Listeners” tab. Edit your configuration so you have HTTP:80 -> HTTP:80 and HTTPS:443 -> HTTP:80.

Don’t forget to check your health check. We’re going to shortly be doing a redirect which will return a 301 HTTP status code. If you use the HTTP ping protocol on port 80, your health check will fail as it’s expecting a 200 response. TCP:80 will work fine though.

The SSL certificate should now be available on your website, but we need to do some configuring before it’s ready to run on your production servers.

As we added SSL to the load balancer, not our instance, we don’t have to deal with configuring keys or listening on a new port. As far as the web server is concerned we’re still running under HTTP which comes with it’s own problems. Luckily AWS are one step ahead and have a (header we can use)[https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/x-forwarded-headers.html] for this purpose.

```
server {
  listen 80;
  server_name www.uvd.co.uk;
  root /path/to/web/dir;

  index index.php;

  proxy_set_header X-Forwarded-Proto $scheme;
  if ( $http_x_forwarded_proto != 'https' ) {
    return 301 https://$host$request_uri;
  }

  location ~ \.php$ {
      # PHP conf
  }
}
```

You may also have to edit settings in your application to tell it you’re using HTTPS. This will likely be in a config file or a setting in a database.

You’ll most likely have some insecure content warnings now. You can’t load insecure content over HTTPS so make sure that you aren’t loading any images or scripts over HTTP. Google Developers have a good [guide](https://web.dev/articles/fixing-mixed-content) on this so give it a read if you want to learn more.

And that’s it. You should should now have a working website behind HTTPS and the best thing is, it doesn’t require maintenance like traditional SSL certificates. There’s no need to renew as AWS does it all automatically for you approximately once a year.