---
layout: post
title: Crash Course Networking
description: "Crash Course Networking"
excerpt_separator: <!--more-->
modified: 2017-01-04
tags: [infrastructure@localhost, operations, networking, TCP, IP, HTTP]
categories: [operations]
---

So computer networking is pretty simple right? It's nothing more than bits of 1's and 0's moving from one computer to another. Except it's actually about electromagnetic waves propagating from one point to another through some medium. Except electromagnetic waves are made up of photons, so what we really have to talk about is quantum mechanics. And this continues forever if we don't set up some abstractions. Get used to that word: abstraction. It's a powerful tool so that we won't have to know the intricacies of an internal combustion engine in order to learn how to drive a car-metaphorically speaking of course. We're learning about networking, not cars!  

<!--more-->

Thankfully the powers that be at the International Organization for Standardization came up with the Open Systems Interconnection model, or OSI model, to break down networking in different layers. We're gonna focus on 3 layers: Application, Transport, and Network. Specifically, we'll look at the interactions between a computer and a web server. I'll be using a textbook called Computer Networking: A Top Down Approach, 6th edition by Kurose and Ross as reference. I'm not using it because it's a particularly good book or anything; it just happens to be the only networking book I have because CS 118 at UCLA uses it. Let's get to it.  

![OSI-Model]({{ site.url }}/images/OSI-Model.png)  

<div id="HTTP"></div>
## The Application Layer ##  

When people talk about going on the "internet", what they usually mean is: going on their web browser and using HTTP, that's hypertext transfer protocol, to look at cute pictures of cats or something. HTTP is an application layer protocol that dictates how a web server and a web browser communicate with one another. Here's an example of what your browser sends to my web server when you visit my page.  
> GET / HTTP/1.1  
> Host: www.dvn7035.com  
> User-Agent: curl/7.47.0  
> Accept: \*/\*  

The web server will respond with a similar header followed by the actual contents of the page.  

> HTTP/1.1 200 OK  
> Server: nginx/1.10.2  
> Date: Wed, 04 Jan 2017 09:05:43 GMT  
> Content-Type: text/html  
> Content-Length: 7614  
> Last-Modified: Wed, 04 Jan 2017 04:10:08 GMT  
> Connection: keep-alive  
> *(continues on with more header information and eventually the text of the actual page)*  

But HTTP is just one protocol at this layer. It's why the web (HTTP) is NOT the internet; it's just a small part of it. On the internet there's also DNS so you can get the IP address of a web server, IMAP for email, FTP for file transfer, etc. But we can't stop here, we gotta go deeper.  

## The Transport Layer ##

Let's go back to our example where your browser requested a page from my web server. How did my web server know that you sent them a request? Where is the web server looking for this request? Where is your web browser looking for these responses?  

First of all, I have to talk about TCP. It stands for transmission control protocol and is widely used at the transport layer. When your web browser wants to send that HTTP request from earlier to my web server, your OS crafts a TCP segment and embeds the HTTP request in the segment's Data field. My web server embeds its response in a TCP segment as well. It's clear the the HTTP piggybacks off of TCP and it will be a recurring design as we go further.  

Here is what a TCP segment looks like.  

![TCP-Segment]({{ site.url }}/images/TCP-Segment.png)  

So what are port numbers? Port numbers are unsigned 16 bit numbers that allows your OS to identify which segment is meant for which process in your computer. Suppose you had a web browser open and it was listening in at port 12345 and it was going to send out that TCP enveloped HTTP request. Your OS would set the Source Port as 12345 and Destination Port as 80, the standard port that web server programs listen in on, and send it to my web server. When my web server replies with its TCP enveloped HTTP response, the ports are switched. Its Source port is 80 and its Destination port is 12345. Your OS will know that the segment was meant for your web browser because the segment's Destination Port is 12345.  

## The Network Layer ##

Now let's talk about IP or internet protocol. Wait you might ask: why do we need yet another protocol? Well with that TCP packet and embedded HTTP request to dvn7035.com, we still don't know where to send it to. We don't have an address, but thankfully the internet protocol solves this problem.  

Every computer and even our home router has an IP address and sends data to each other in the form of IP packets. They look something like this.  

![IP-Packet]({{ site.url }}/images/IP-Packet.png)  

Let's ignore every field except Source Address, Destination Address, and Data. Let's say our address is 192.168.1.2 and my web server's address is 162.243.149.160. Your OS again does the heavy lifting and embeds the TCP segment with the HTTP request in the IP packet's Data field with Source Address 192.168.1.2 and Destination Address 162.243.149.160 and sends it to your router. My web server's response comes to your computer as a packet with the Source and Destination Address reversed (just like the TCP ports).  

## In Summary ##

The interconnected network of networks of computers connected via this TCP/IP suite is called the internet. However, TCP/IP is not how modern applications use the internet. They use higher abstractions at the application layer. Looking at our example of visiting this website, your computer and my web server talk to each other with an HTTP request encapsulated in a TCP segment that is further encapsulated in an IP packet. Here's another infographic for your benefit.  

![OSI-Overview]({{ site.url }}/images/OSI-Overview.png)  

Argh, enough with this theory stuff. Let's fire up some servers already! I feel ya. Our next topic will be how to set up your first web server.
