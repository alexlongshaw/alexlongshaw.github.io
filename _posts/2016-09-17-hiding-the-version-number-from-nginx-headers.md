---
layout: post
title:  "Hiding the version number from NGINX headers"
date:   2016-09-17 17:28:00 +0000
categories: security
---

### So, what is “hardening” exactly?

Hardening is the process of limiting the attack surface of your application and the supporting services. If you want to 
learn more about what an attack surface is, [OWASP](https://www.owasp.org/index.php/Attack_Surface_Analysis_Cheat_Sheet){:target="_blank" rel="noopener"} 
have some great resources.

The vast majority of websites are hackable with enough resources and attacks can come in many forms. This post will cover 
the basics of protecting your server from known vulnerabilities within NGINX. If an exploit in software is discovered 
before the developers or security researchers find it, it is known as a “zero day vulnerability”. These are much harder 
to protect yourself from and so won't be covered here.

Known vulnerabilities are much easier to protect yourself from, but surprisingly “44% of breaches comes from 
vulnerabilities which are two to four years old”[1]. We can minimise this risk by keeping software up to date and hiding 
the version numbers of software, such as NGINX.

### Finding the version number

There are a number of ways you can find version numbers so we’re just going to look at [curl](https://curl.haxx.se/){:target="_blank" rel="noopener"} in 
the command line of a *NIX environment (Mac OSX/Linux). Other methods you could use include sending HTTP requests to the 
website in something like [Postman](https://www.getpostman.com/) or by using a tool such as [Wappalyzer](https://wappalyzer.com/){:target="_blank" rel="noopener"}, which is 
available as a browser extension for Chrome and Firefox.

```curl --head http://www.example.com```

If we run a curl command with the head flag set, like the above, we will get the headers of the website in question,
similar to the output below. In this case, if we look at the “Server”, section, we can see that the website is running 
NGINX version 1.1.9.

```
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8
Date: Fri, 16 Sep 2016 00:09:32 GMT
Server: nginx/1.1.19
Connection: keep-alive
```

### What are the risks?

Now our attacker knows the version of NGINX, they can look through [known vulnerabilities](http://nginx.org/en/security_advisories.html){:target="_blank" rel="noopener"} 
that they can use to cause denial of service or get access to potentially sensitive information.

If we look at the known vulnerabilities in this version, we can see the following potential medium severity risks. 

- Invalid pointer dereference in resolver
- Use-after-free during CNAME response processing in resolver
- Insufficient limits of CNAME resolution in resolver
- SSL session reuse vulnerability
- Request line parsing vulnerability
- Memory disclosure with specially crafted HTTP backend responses

This gives the attacker a big head start for not a lot of work.

Thankfully, it is easy to remove the version number from being displayed. All we need to do it set “server_tokens” to 
off in the nginx config.

```
# /etc/nginx/nginx.conf

http {
    # Your existing content here
    
    server_tokens off;
}
```

Note, you'll also need to restart NGINX before the changes come into effect.

Now, if we run our curl command again, we'll see that the version number has disappeared from the headers. Removing the 
“nginx” server name is also possible, but it is a bit harder and requires changing some code and recompiling NGINX from 
source. 

```
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8
Date: Fri, 16 Sep 2016 00:09:39 GMT
Server: nginx
Connection: keep-alive
```

We've made a good start here and for a small effort on our part, we've made this method of attack quite a bit harder. 
However, all we are providing here is security by obscurity, which is security in its weakest form. If your site is 
being targeted, the attacker might wait until a vulnerability has been published and immediately attack, 
so it is still important to update software as soon as security patches become available. 

---

[1] [http://www.theregister.co.uk/](http://www.theregister.co.uk/2015/02/23/hp_hack_vulnerable_threat_study/){:target="_blank" rel="noopener"}
