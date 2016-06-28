---
layout: post
title:  "Event Sourcing with eventuate and play!"
date:   2016-06-28 10:20:30 +0800
categories: play framework eventuate scala akka
---


Hi, in this post we'll try and create a simple trade manager application.

Background - 
I have worked mostly for Investment Banks and there we have a common use case of a trade manager which accepts trades and updates on it and persists them locally for analysis or settlements or reporting.
So we also frequently get asked in interviews to design such a system.
This post will show you how easy it is to implement a asynchronous event-sourced app with Play and eventuate. 

TL;DR, 
here is the source - [GitHub](https://github.com/kunalkanojia/react-play-eventsourcing )

Architecture Overview -  

//TODO
