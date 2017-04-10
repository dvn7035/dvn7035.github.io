---
layout: post
title: New Raspberry Pi 3s
description: "New Raspberry Pi 3s"
excerpt_separator: <!--more-->
modified: 2017-04-07
tags: [infrastructure@localhost, server, raspberry, pi]
categories: [operations]
---

![digital_ocean_1]({{ site.url }}/images/raspberry_pi.png)
<!--more-->
###### 3 Raspberry Pi 3's stacked on each other and a Raspberry Pi Model B for scale ######

Over the spring break, I was able to get my hands on 3 new Raspberry Pi 3's. They're going to be my
new homelab that will run my web site and a few other projects I have in store.  

The Raspberry Pis are pretty interesting pieces of hardware. They run power efficient ARM
processors, are dirt cheap, and best of all can run a Linux distribution. Sure they might not be
that powerful, but I enjoy the challenge of working within the bounds of some constraint. You don't
always get to play with the greatest and shiniest of toys.  

In the picture above, is my Raspberry Pi Model B and the 3 new Pis stacked on each other. The Model
B will be hooked up to a HDD via USB and will run LDAP, Salt, NFS, and other services that are
stateful (I wonder if it is strong enough to run GitLab). Meanwhile the new Pis will be part of a
Docker Swarm cluster. The projects that I can containerize, will run on that cluster and Swarm will
manage balancing the load on each Pi and handle failover. Stay tuned for write ups about that.
