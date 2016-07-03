---
layout: post
title:  "Event Sourcing with play framework and eventuate (Part 1 - The design)"
date:   2016-06-28 10:20:30 +0800
categories: play framework eventuate scala akka
---


Hi, in this post series we'll create a simple trade manager application.

We will see how easy it is to implement asynchronous event-sourced app with Play and eventuate. 
I could have used Akka persistence as well, maybe in another post I'll change this to use AP and then compare it with Eventuate. 
For now read here how they compare here - [Martins Blog](http://krasserm.github.io/2015/05/25/akka-persistence-eventuate-comparison/)

**TL;DR**, 
here is the source - [GitHub](https://github.com/kunalkanojia/react-play-eventsourcing )

**Architecture Overview** -  

The architecture of the application looks like this - 

![Application Architecture](/images/architecture.png){: .center}

Its our standard architecture, but with one change. We have replaced ORM with [Eventuate](http://rbmhtechnology.github.io/eventuate/). 


**Actor system** - 

The actor system hierarchy for our app will be structured like below - 

![Actor System Hierarchy](/images/actor_system.png)

_UserManager_ :
Extends from [Event Sourced View](http://rbmhtechnology.github.io/eventuate/architecture.html#event-sourced-views). One per system. Created when the application starts. Stores all users in memory and routes all create and read message to respective UserActor.

_UserActor_ :
Extends from [Event Sourced Actor](http://rbmhtechnology.github.io/eventuate/user-guide.html#event-sourced-actors). One per user in the system. Created when a user signs up. Persists user events to the event log.

_TradeManager_ :
 Extends from [Event Sourced View](http://rbmhtechnology.github.io/eventuate/architecture.html#event-sourced-views). One per user in the system. Created when the user signs up from the UserActor Stores all trades in memory. Routes all trade create and update message to respective trade actor.

_TradeActor_ :
Extends from [Event Sourced Actor](http://rbmhtechnology.github.io/eventuate/user-guide.html#event-sourced-actors). One per trade. Created by trade manager when trade create command is received. Receives all trade create and update commands from the TradeManager and persists to the event log.

_TradeViewAggregateActor_ - 
Extends from [Event Sourced View](http://rbmhtechnology.github.io/eventuate/architecture.html#event-sourced-views). One per system. Created on application startup. Receives command from all trade actors irrespective of the user and sends message to connected Web Socket user actors.


**Heroku Deployment** - 

The app is deployed on heroku, you can access it here - [http://play-eventsourcing.herokuapp.com/](http://play-eventsourcing.herokuapp.com/)


**References**: 

[http://martinfowler.com/eaaDev/EventSourcing.html](http://martinfowler.com/eaaDev/EventSourcing.html)

[http://rbmhtechnology.github.io/eventuate/](http://rbmhtechnology.github.io/eventuate/)

[https://www.playframework.com/](https://www.playframework.com/)

