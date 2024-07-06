---
layout: post
title:  "How we do remote pair programming"
date:   2016-11-07 12:30:34 +0000
categories: pair-programming
disqus_id: 8c5b58c1-bbef-417c-aacd-05837434272f
---

*This post was originally available at: https://www.uvd.co.uk/blog/remote-pair-programming/*

We’ve recently moved to flexitime at UVD, so we’ve been waiting for a good time to try remote pair programming. We
thought we’d share how we do it. We hosted a user story mapping workshop recently with seven attendees, as well as the
UVD team, so Jay and me offered to work from home for the two days to make space, and also to work distraction free. As
we’ve been pair programming on Limpid Markets, we wanted to find a way to carry on in the two days we weren’t in the
office. Living at opposite ends of London, it wasn’t practical for us to meet up, unless we ended up somewhere near the
office anyway so we had a look into some remote pair programming options.

We looked at a few technologies such as [Screenhero](https://screenhero.com/) by Slack and the
[AtomPair](https://blog.pusher.com/atom-pair/) plugin but these either came at a price or involved some set up. Since
we’re both using MacBook Pros we decided to use the built in screen sharing app which allows one user to be the host and
the other to remote in so we could keep using the tools we’re used to.

### Working with Screen Sharing

To setup screen sharing on Mac, the host user has to enable it. You can do this by going to System Preferences > Sharing
and then ticking “Screen Sharing” and selecting “All users”.

Once this is done, the guest user can open up the Screen Sharing app. You can easily find this by opening Spotlight
(cmd + space) and searching for “Screen Sharing”. Selecting this opens the app which allows you to enter the hostname
or Apple ID of the user you wan to connect to. Simply type in their Apple ID (the email address of their Apple account)
and connect.

We found it to be a surprisingly painless process. The connection was almost instantaneous and there wasn’t much in the
way of bad connections, although we were both on high speed Virgin fibre. Audio quality was good, better than we
generally experience with video calling services and video quality was excellent, mostly appearing like I was on my own
computer, with the occasional moments of fuzziness and a small amount of lag.

There were only a few minor downsides; one being the fixed screen resolution. Jay’s 15 inch laptop screen was too small
scaled down to my 13 inch screen but also had a large border around my 24 inch monitor. There were also some issues
with mouse control. Gestures didn’t work and nor would the Mission Control key work on the host, so managing multiple
desktops was a bit harder than normal. We found it easier using a single desktop and clicking on icons in the dock or
using spotlight to change windows. Another minor complaint was that mouse scrolling with the wheel was at an almost
standstill, but was easy enough to workaround by dragging the scroll bar or sometimes with me just asking Jay to do the
scrolling.

### Remote pair programming

#### The pros

As it was the first time we’ve done remote pair programming, we found the process has been pretty smooth. I think it
worked well as we were both working from home so we weren’t a distraction to other staff. We were also free from
answering the office phone which normally puts pairing at a standstill whilst one deals with the call.

Part of the success was due to us already having a good communication process. We use Slack for team chat which has been
essential. It’s important to be aware you’re not visible to the team, so our presence on Slack ensured that other staff
knew we were available to test and setup QA sessions as items on the Kanban board were pulled through.

#### The cons

Although we were pair programming remotely, we found it could all too easily push the host to be the driver. Using
something like a [pomodora timer](https://www.marinaratimer.com/) would have been helpful to keep us on track.

We also had some questions for Eddy, our lead developer. We got through okay asking questions on Slack and showing
commits on BitBucket and but harder issues might have been a problem. Usually he would come to my desk and we could
look through more difficult problems together.

Finally, we did have a backup plan for worst case. If we had technical issues our plan was for me to carry on with the
Limpid Markets work and Jay to add a new feature to Botty McBotface, our helpful bot that organises our days off, but
luckily we didn’t have to resort to that. We also got some good solutions from computer repair service and we were able
to avoid screen sharing issues.

We’re planning on doing this again soon. To avoid some of the screen sharing issues we are going to try using a mixture
of screen sharing and using AtomPair. It’s not either of our main IDEs but we feel the benefits are worth trying for a
day. We’ll update this blog post with how we got on.