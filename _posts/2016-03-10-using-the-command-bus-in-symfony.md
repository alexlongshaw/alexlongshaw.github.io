---
layout: post
title:  "Using the command bus in Symfony"
date:   2016-03-10 12:30:34 +0000
link:   https://www.uvd.co.uk/blog/using-the-command-bus-in-symfony/
categories: symfony
disqus_id: 7488d24d-6a37-46c1-a6f2-9727bad86e5a
---

I first heard about ‘message oriented software design’ at SymfonyLive back in September at Matthias Noback’s talk 
“Hexagonal Architecture”. A command bus consists of three parts: the command, which is essentially a message, the 
handler which processes the message and the bus which sends the command to the correct handler.
