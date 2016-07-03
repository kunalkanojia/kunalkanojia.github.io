---
layout: post
title:  "Event Sourcing with eventuate and play!"
date:   2016-06-28 10:20:30 +0800
categories: play framework eventuate scala akka
---


Hi, in this post we'll try and create a simple trade manager application.

We will see how easy it is to implement asynchronous event-sourced app with Play and eventuate. 
I could have used Akka persistence as well, maybe in another post I'll change this to AP and then compare it with Eventuate. 
For now read here how they compare - [Martins Blog](http://krasserm.github.io/2015/05/25/akka-persistence-eventuate-comparison/)

TL;DR, 
here is the source - [GitHub](https://github.com/kunalkanojia/react-play-eventsourcing )

Architecture Overview -  

The architecture of the application looks like this - 

![Application Architecture](/images/architecture.png)

Its our standard architecture, but with one change. We have replaced ORM with eventuate. 


Actor system Hierarchy - 
The actor system for our app will be structured like below - 

![Actor System Hierarchy](/images/actor_system.png)

Manager Actors  - 


//TODO
