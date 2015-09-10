+++
categories = ["Development"]
date = "2015-09-09T07:57:38+01:00"
description = "Experiences of remote pair programming and comparisons of tools for collaborative editing"
draft = true
tags = ["development", "agile", "pair programming", "collaboration", "distributed/remote working"]
title = "Remote pair programming"

+++

During a previous job I spent a lot of time working with delivery teams on other continents, helping them develop software. I was lucky enough to visit them on several occassions for a week at a time, and whilst I was there made lots of progress working with the on-site developers.  Unfortunatley I was not able to stay on-site for the duration of the project and so needed to find other ways of collaborating with the teams remotely from back in the UK.

We started with daily phone calls but we quickly found this to be an awful experience.  Over the phone we struggled to hear what the team members were saying, pick up body language or subtle nuances of the discussion or even pick out who was saying what.  We upgraded to video conferences for the meetings but found, whilst considerably better than the phone, this was still not effective.  We found that, rather ironically, when you are working remotely you need to work even more collaboratively.  For example, at the meetings we would discuss approaches for the day ahead but things would get lost or misinterpreted and following the meeting, people would then go away and diligently write lots of code that was taking us off in different directions.  Communicating over a phone or even a video conference is not nearly as effective as in person.  

We quickly looked to augment the video conferences with more hands-on collaborative working and started investigating tools which might support remote pair programming.

### Tools

There are lots of tools available that can help support remote pairing and we tried a lot of them.  In my opinion, these tools can be divided into 6 distinct categories as follows:

1. [Video Conferencing with screen share]({{< ref "#1-video-conferencing-with-screen-share" >}})
2. [VNC]({{< ref "#2-vnc" >}})
3. [TMUX]({{< ref "#3-tmux" >}})
4. [Cloud based editors]({{< ref "#4-cloud-based-editors" >}})
5. [Cloud based development environments]({{< ref "#5-cloud-based-development-environments" >}})
6. [Interactive screen share]({{< ref "#6-interactive-screen-share" >}})

#### 1. Video Conferencing with screen share

Whilst looking for pair programming tools, I came across lots of accounts of people claiming success using screen sharing tools like [Skype](http://www.skype.com), [Google Hangouts](http://hangouts.google.com) and [join.me](http://www.join.me).  This category of tools were originally designed as communication tools to support video conferencing and screen sharing.  The screen sharing within these tools is uni-directional so was probably originally intended for demonstrating or presenting something from the host's machine.

In practice, we found that these tools were okay for demonstrating or presenting concepts, especially to larger groups of people, but did not work well for collaborative editing and pairing.  Only one person could edit or navigate at once and transferring control required switching to the other person's machine and for them to pull changes made on the original machine.

#### 2. VNC

Virtual Network Computing is a system for the sharing and remote control of desktops.  It transmits keyboard and mouse events to the host and the graphical screen updates back the other way.  VNC is platform independent and there are clients available for most platforms like [RealVNC](https://www.realvnc.com/) and [Chicken of the VNC](http://sourceforge.net/projects/cotvnc/).

We were initially very excited by VNC because it promised everything we wanted - being able to share a desktop and allow multiple people to collaboratively edit the source code using graphical desktop based IDEs that people were already familiar with.  Unfortunately, in practice we found VNC was simply too slow.  The lag between pressing a key and the character appearing on screen was extremely noticeable and we kept running into problems where one user was typing at the same time as another user moving the cursor to a different part of the screen.  We tried taking it in turns to edit and navigate so only one person was actively editing at once and then explicitly transferring control to the other person when they wanted it.  Althought this helped allieviate some of the problems, the latency VNC was adding was still really irritating and ultimately slowing down our work.

#### 3. TMUX

#### 4. Cloud based editors
Cloud9, Atom, Ace

#### 5. Cloud based development environments

#### 6. Interactive screen share
ScreenHero, Floobits, Nitrous.io

### General experiences

User with higher network bandwidth to host sharing session.

Use combinations of tools e.g. VC for discussions, VC and screen sharing for demoing especially to larger groups, Collaborative editing tools for pairing and use lots of communication tools like chat all the time.

Email is awful

Just because there is a big distance (and potentially time zones) between you doesn't mean you should batch up communication.  The greater the distance the more important it is to have frequent small communication (chat applications work really well for this).