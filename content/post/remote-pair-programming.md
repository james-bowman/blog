+++
categories = ["Development"]
date = "2015-09-09T07:57:38+01:00"
description = "Experiences of remote pair programming and comparisons of tools for collaborative editing"
draft = false
tags = ["development", "agile", "pair programming", "collaboration", "distributed/remote working"]
title = "Remote pair programming"

+++

During a previous job I spent a lot of time working with delivery teams on other continents, helping them develop software. I was lucky enough to visit them on several occassions for a week at a time, and whilst I was there made lots of progress working with the on-site developers.  Unfortunatley I was not able to stay on-site for the duration of the project and so needed to find other ways of collaborating with the teams remotely from back in the UK.

We started with daily phone calls but we quickly found this to be an awful experience.  Over the phone we struggled to hear what the team members were saying, pick up body language or subtle nuances of the discussion or even pick out who was saying what.  We upgraded to video conferences for the meetings but found, whilst considerably better than the phone, this was still not effective.  We found that, rather ironically, when you are working remotely you need to work even more collaboratively.  For example, at the meetings we would discuss approaches for the day ahead but things would get lost or misinterpreted and following the meeting, people would then go away and diligently write lots of code that was taking us off in different directions.  Communicating over a phone or even a video conference is not nearly as effective as in person.  

We quickly looked to augment the video conferences with more hands-on collaborative working and started investigating tools which might support remote pair programming.

### Tools

There are lots of tools available that can help support remote pairing and we tried a lot of them.  In my opinion, these tools can be divided into 6 distinct categories as follows:

1. [Video Conferencing with screen share]({{< ref "#1-video-conferencing-with-screen-share" >}})
2. [TMUX]({{< ref "#2-tmux" >}})
3. [VNC]({{< ref "#3-vnc" >}})
4. [Cloud hosted TMUX or VNC]({{< ref "#4-cloud-hosted-tmux-or-vnc" >}})
5. [Web based IDEs]({{< ref "#5-web-based-ides" >}})
6. [Desktop IDEs]({{< ref "#6-desktop-ides" >}})

#### 1. Video Conferencing with screen share

Whilst looking for pair programming tools, I came across lots of accounts of people claiming success using screen sharing tools like [Skype](http://www.skype.com), [Google Hangouts](http://hangouts.google.com) and [join.me](http://www.join.me).  This category of tools were originally designed as communication tools to support video conferencing and screen sharing.  The screen sharing within these tools is uni-directional and supports demonstrating or presenting something from the host's machine.

In practice, we found that these tools were okay for demonstrating or presenting concepts, especially to larger groups of people, but did not work well for collaborative editing and pairing.  Only one person could edit or navigate at once and transferring control required switching to the other person's machine and for them to pull changes made on the original machine.

#### 2. TMUX

Terminal multiplexing can allow multiple users to simultaneously connect to the same terminal session.  Using TMUX, terminal based editors like vi and emacs can be used simultaneously by multiple clients allowing collaborative editing.

Unfortunately, the developers we were working with were used to Windows and graphical user interfaces.  They had experience with, and were productive on, GUI based IDEs specifically IntelliJ and Eclipse.  Using terminal based editors with TMUX represented a steep learning curve and short-term drop in productivity for the developers and would have been a hard sell.  Additionally, we also found challenges sharing sessions with TMUX across corporate firewalls.

#### 3. VNC

Virtual Network Computing is a system for the sharing and remote control of desktops.  It transmits keyboard and mouse events to the host and the graphical screen updates back the other way.  VNC is platform independent and there are clients available for most platforms like [RealVNC](https://www.realvnc.com/) and [Chicken of the VNC](http://sourceforge.net/projects/cotvnc/).

We were initially very excited by VNC because it promised everything we wanted - being able to share a desktop and allow multiple people to collaboratively edit the source code using graphical desktop based IDEs that people were already familiar with.  Unfortunately, in practice we found VNC was simply too slow.  The lag between pressing a key and the character appearing on screen was extremely noticeable and we kept running into problems where one user was typing at the same time as another user moving the cursor to a different part of the screen.  We tried taking it in turns to edit and navigate so only one person was actively editing at once and then explicitly transferring control to the other person when they wanted it.  Althought this helped allieviate some of the problems, the latency VNC was adding was still really irritating and ultimately slowing down our work. Furthermore, we found that the latency was not the same for both users - the person hosting the session experienced significantly lower latency than the guest.  The natural tendency was to attribute the apparent slowness to the person they were pairing with rather than the network latency which led to frustration and tension between pairs.  Like TMUX, we also found challenges using VNC across corporate firewalls.

#### 4. Cloud hosted TMUX or VNC

This category builds on technologies like TMUX and VNC to share sessions but rather than using direct peer-to-peer connectivity between clients, clients all connect to a centralised host.  Examples of tools available to help support this model of working are [Remote Pair Chef](https://github.com/rondale-sc/remote_pair_chef) and [Syme](https://syme.herokuapp.com/).  The advantages of this approach over simply using TMUX or VNC with one party hosting the session are as follows:

- It works much better with corporate firewalls as all clients connect to an external host exposed on the internet rather than trying to connect to a host within a corporate network.

- By locating the host in a suitable geographical location, all clients can experience similar latency and so tend to be more understanding when working together.

- The development environment is truly shared, rather than being someone's personal workspace.

Unfortunately we found that although moving the host into the cloud helped with a lot of the challenges we were facing, TMUX with text based editors was still a hard sell for the developers and VNC was still too slow.  Also, this might have meant moving the source code into the cloud which was not an option available to us.

#### 5. Web based IDEs

This category of tool are rich web application based editors and IDEs accessed via a web browser.  Examples of this type of tool include [Cloud9](https://c9.io/), [Atom](https://atom.io/) and [MadEye](https://madeye.io/).  In a similar way to Google Docs, they support collaborative editing by multiple users simultaneously.  

We found these IDEs really exciting.  They use HTTP and clients connect to a central host accessible via the internet so avoid the connectivity and firewall based challenges that impacted options in some of the other categories.  Some of these options combine voice communication with the collaborative editing and even execution environments for building and testing code directly from the IDE.  Unfortunately at the time we considered these options, most offerings were still relatively basic in terms of the features they offered so were a hard sell for developers coming from desktop IDEs.  Also, these options tend to rely on storing the source code in the cloud, so the editor can access it.  This was not an option available to us.


#### 6. Desktop IDEs

Like VNC, this category of tools promised everything we wanted, but unlike VNC, the tools in this category actually delivered!  We wanted to be able to share desktops and allow multiple people to collaboratively edit the source code using the graphical desktop based IDEs they were already familiar with.  Examples of tools in this category are [ScreenHero](https://screenhero.com/), [Floobits](https://floobits.com/) and [Nitrous.io](https://www.nitrous.io/).  All three of these examples require a client to be installed on everyone's desktop but work in very different ways.  Nitrous.io provides a web based IDE but, with a paid plan, can also continuously synchronise changes with the local filesystem on the user's desktop allowing them to edit files using an IDE of their choice.  The Floobits client integrates with a small set of desktop IDEs to synchronise changes with remote users in the IDE directly.  Floobits currently supports NeoVim, Emacs, IntelliJ and Sublime Text IDEs.  ScreenHero works in a similar way to VNC and transfers keyboard, mouse and screen events and voice between 2 or more users.  

We found ScreenHero the easiest to use and the most flexible as it allowed multiple people to collaboratively interact with the same desktop and use any applications on that desktop not just IDEs.  ScreenHero could be used to collaboratively edit word documents or presentations.  It seemed to adjust well to differnt bandwidth constraints and degraded relatively gracefully when resources were constrained.

### General experiences

- Email is awful

- The person with higher network bandwidth should host the sharing session when using tools like Google hangouts, VNC, ScreenHero, etc.

- Use combinations of tools for different purposes e.g. Video Conferencing for discussions, Video Conferencing and screen sharing for demoing especially to larger groups, Collaborative editing tools for pairing and use communication tools like chat *all the time*.

- Just because there is a big distance (and potentially time zones) between you doesn't mean you should batch up communication.  The greater the distance the more important it is to have frequent short communications (chat applications work really well for this).