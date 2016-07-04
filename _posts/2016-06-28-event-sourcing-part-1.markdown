---
layout: page
title:  "Event Sourcing with scala, play framework and eventuate (Part 1 - Introduction)"
date:   2016-06-28 10:20:30 +0800
quote: Just like in real life, in code, a feature is triggered by an action (command) that results in a sequence of events that might or might not cause side effects.  ― Greg Young, Exploring CQRS and Event Sourcing
comments: true
---


Hi, in this post series we'll create an event-sourced application from ground up. The application is going to be a over-simplified trade manager where a user can login, create trades and edit trades. It is a common use case in banks where applications receives trades and corrections and does some sort of analysis or reporting from it.

### Event Sourcing

First things first, what do we mean by event sourcing. This is how Martin Fowler defines it -
<div class="message"> Event Sourcing ensures that all changes to application state are stored as a sequence of events. Not just can we query these events, we can also use the event log to reconstruct past states, and as a foundation to automatically adjust the state to cope with retroactive changes. </div>

So our application is going to store application state as a sequence of events. These events will be persisted in an event log (Level DB in our case) and will be replayed to recover application state.

Events are immutable facts that are only ever appended to an event log which allows for very high transaction rates and efficient replication. 

This is very different from the CRUD model most of us are used to building. A lot has been spoken about the pros and cons of each of these models so I'll not go into that. I have shared some links in the end where you can read more.

### Eventuate

> Eventuate is a toolkit for building applications composed of event-driven and event-sourced services that collaborate by exchanging events over shared event logs. Services can either be co-located on a single node or distributed up to global scale.

We will see how easy it is to implement asynchronous event-sourced app with Play and eventuate. 
I could have used Akka persistence as well, maybe in another post I'll change this to use AP and then compare it with Eventuate. 
For now read here how they compare here - [Martins Blog](http://krasserm.github.io/2015/05/25/akka-persistence-eventuate-comparison/)

_Another good thing I would like to point out about eventuate is - Martin and his team is very helpful. Whenever we post some doubts or problems on eventuate gitter channel we always get quick response from Martin._

### tl;dr
If you are don't want to read more and dive straight into the code, then grab it here - [GitHub](https://github.com/kunalkanojia/react-play-eventsourcing )


### Architecture Overview 

The architecture of the application looks like this - 

{% include image.html url="/images/architecture.png" style="text-align: center;" description="Its our standard architecture, but with one change. We have replaced ORM with Eventuate." %}


Every user request received by Play framework controller which then fires a Command to Eventuate actors. The persistent actor will persist event to the event log and respond with success or failure message. 

This whole cycle will be completely non blocking and event driven. We will soon see how.


### Sequence 

The basic flow of our application is going to be this -
 
1. Controller receives a request from UI

2. Controller will send data to service.

3. Service will then send a Command to a Manager. This manager will be a [EventSourcedView](http://rbmhtechnology.github.io/eventuate/reference/event-sourcing.html#event-sourced-views).

4. Manager will delegate the persistence commands to the persistent actor which are an instance of [EventSourcedActor](http://rbmhtechnology.github.io/eventuate/reference/event-sourcing.html#event-sourced-actors).

5. Depending on successful persistence the persistent actor will send a success or a failure response. It also informs the manager asynchronously to update its state so that retrieval can be done form the Manager.

<strong>Models</strong>

We are going to have two simple models in the system, User and Trade.
<script src="https://gist.github.com/kunalkanojia/d5e0b3874d84a4c6e4664b361d22ceed.js"></script>
<script src="https://gist.github.com/kunalkanojia/664ec1bbbbbfb990c4b9f896711c073b.js"></script>

<strong>Trade Commands, Replies & Events</strong>

<script src="https://gist.github.com/kunalkanojia/9e716281e2136e0677e7517269b35053.js"></script>

For example, When user creates a trade form the UI.

A `CreateTrade` command is sent to the Actor which will persist a `TradeCreated` event to the event log and respond with `CreateTradeSuccess`.

### Actor system

If you don't know much about akka and the actor system, you can read here [http://doc.akka.io/docs/akka/2.4.7/general/actor-systems.html](http://doc.akka.io/docs/akka/2.4.7/general/actor-systems.html)

The actor system hierarchy for our app will be structured like below - 

{% include image.html url="/images/actor_system.png" style="text-align: center;" %}

Below is the description of what role each actor plays in the system.

_UserManager_ :
Extends from [EventSourcedView](http://rbmhtechnology.github.io/eventuate/architecture.html#event-sourced-views). One per system. Created when the application starts. Stores all users in memory and routes all create and read message to respective UserActor.

_UserActor_ :
Extends from [EventSourcedActor](http://rbmhtechnology.github.io/eventuate/user-guide.html#event-sourced-actors). One per user in the system. Created when a user signs up. Persists user events to the event log.

_TradeManager_ :
 Extends from [EventSourcedView](http://rbmhtechnology.github.io/eventuate/architecture.html#event-sourced-views). One per user in the system. Created when the user signs up from the UserActor Stores all trades in memory. Routes all trade create and update message to respective trade actor.

_TradeActor_ :
Extends from [EventSourcedActor](http://rbmhtechnology.github.io/eventuate/user-guide.html#event-sourced-actors). One per trade. Created by trade manager when trade create command is received. Receives all trade create and update commands from the TradeManager and persists to the event log.

_TradeViewAggregateActor_ :
Extends from [EventSourcedView](http://rbmhtechnology.github.io/eventuate/architecture.html#event-sourced-views). One per system. Created on application startup. Receives command from all trade actors irrespective of the user and sends message to connected Web Socket user actors.


### Whats Next - 

We will follow TDD and start implementing application. This should also give you a good idea on how to test asynchronous apps written using akka. 

### References

- [http://codebetter.com/gregyoung/2010/02/20/why-use-event-sourcing/](http://codebetter.com/gregyoung/2010/02/20/why-use-event-sourcing/)

- [http://martinfowler.com/eaaDev/EventSourcing.html](http://martinfowler.com/eaaDev/EventSourcing.html)

- [http://rbmhtechnology.github.io/eventuate/](http://rbmhtechnology.github.io/eventuate/)

- [https://www.playframework.com/](https://www.playframework.com/)

