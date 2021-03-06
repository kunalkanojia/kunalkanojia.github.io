---
layout: page
title:  "Event Sourcing with scala, play framework and eventuate (Part 1 - Introduction)"
quote: Just like in real life, in code, a feature is triggered by an action (command) that results in a sequence of events that might or might not cause side effects.  ― Greg Young, Exploring CQRS and Event Sourcing
comments: true
---


Hi, in this series of post we'll create an event-sourced application from ground up. The application is going to be a over-simplified trade manager where a user can login, create trades and edit trades. 

In a way its our usual CRUD application with trade as our domain. But we will implement it using CQRS and event sourcing and see how it differs from CRUD.

### Event Sourcing
 
> Event Sourcing ensures that all changes to application state are stored as a sequence of events. Not just can we query these events, we can also use the event log to reconstruct past states, and as a foundation to automatically adjust the state to cope with retroactive changes. - Martin Fowler

What exactly is event sourcing?

Event Sourcing is a pattern which defines an approach to handle operations on data that is driven by a sequence of events, each of which is recorded in an append-only store. 

Application code sends a series of events that imperatively describe each action that has occurred on the data to the event store, where they are persisted. 

Each event represents a set of changes to the data.
 
The events are persisted in an event store that acts as the source of truth or system of record. Since events are only ever appended to an event log, this allows for very high transaction rates and efficient replication. 

Some benefits we get out of event sourcing are :

  - Complete log of every state change ever
  - Unmatched traceability and debugability
  - Very good performance characteristics due to append only store
  - No more mapping complex objects to tables

Following the above definition our application is going to store trade states as a sequence of events. These events will be persisted in an event log ([Level DB](http://leveldb.org/)) and will be replayed on restart to recover application state. 

Since I have spent most of my career in investment banking `Trade` is the simplest domain I can think of.

In real world events on a typical back-office trade settlement systems would be like this - 

{% include image.html url="/images/trade_events.png" style="text-align: center;" %}

We are only going to look at trade create and update as events in our current implementation. This should be good enough to explain the concepts.

### Eventuate

> Eventuate is a toolkit for building applications composed of event-driven and event-sourced services that collaborate by exchanging events over shared event logs. Services can either be co-located on a single node or distributed up to global scale.

Eventuate provides several abstractions for building event sourced application components. We will be using two of those [EventSourcedView](http://rbmhtechnology.github.io/eventuate/reference/event-sourcing.html#event-sourced-views) and [EventSourcedActor](http://rbmhtechnology.github.io/eventuate/user-guide.html#event-sourced-actors).

We will see how easy it is to implement asynchronous event-sourced app with eventuate. We just need to implement our events and domains. Persistence and replaying of events will be taken care by eventuate provided actors.

But what about Akka Persistence?

Eventuate is similar to akka persistence and I could have used it as well.
Read this post to see comparison - [Martins Blog](http://krasserm.github.io/2015/05/25/akka-persistence-eventuate-comparison/)

To jump to akka persistence example go here - [akka-persistence-sample](https://github.com/kunalkanojia/akka-persitence-sample). It uses the same model and architecture defined in this post.


### tl;dr
If you are don't want to read more and dive straight into the code, then grab it here - [GitHub Repo](https://github.com/kunalkanojia/react-play-eventsourcing )


### Architecture Overview 

The architecture of the application will look like this:

{% include image.html url="/images/architecture.png" style="text-align: center;" description="Its our standard architecture, but with one change. We have replaced ORM with Eventuate." %}


A user request is received by Play framework controller which then fires a Command to Eventuate actors. The persistent actor will persist event to the event log and respond with success or failure message. 

This whole cycle will be completely non blocking and event driven. We will soon see how.


### Sequence 

The basic flow of our application is going to be this:
 
1. Controller receives a request from UI

2. Controller will send data to service.

3. Service will then send a Command to a Manager. This manager will be a [EventSourcedView](http://rbmhtechnology.github.io/eventuate/reference/event-sourcing.html#event-sourced-views).

4. Manager will delegate the persistence commands to the persistent actor which are an instance of [EventSourcedActor](http://rbmhtechnology.github.io/eventuate/reference/event-sourcing.html#event-sourced-actors).

5. Depending on successful persistence the persistent actor will send a success or a failure response. It also informs the manager asynchronously to update its state so that retrieval can be done form the Manager.

<strong>Model</strong>

We are going to have a very simple trade model.

<script src="https://gist.github.com/kunalkanojia/664ec1bbbbbfb990c4b9f896711c073b.js"></script>

<strong>Trade Commands, Replies & Events</strong>

<script src="https://gist.github.com/kunalkanojia/9e716281e2136e0677e7517269b35053.js"></script>

For example, When user creates a trade form the UI.

A `CreateTrade` command is sent to the Actor which will persist a `TradeCreated` event to the event log and respond with `CreateTradeSuccess`.

### Actor system

If you don't know much about akka and the actor system, you can read here [http://doc.akka.io/docs/akka/2.4.7/general/actor-systems.html](http://doc.akka.io/docs/akka/2.4.7/general/actor-systems.html)

The actor system hierarchy for our app will be structured like below: 

{% include image.html url="/images/actor_system.png" style="text-align: center;" %}

Below is the description of what role each actor plays in the system.

_UserManager_ :
Extends from [EventSourcedView](http://rbmhtechnology.github.io/eventuate/architecture.html#event-sourced-views). One per system. Created when the application starts. Stores all users in memory and routes all create and read message to respective UserActor.

_UserActor_ :
Extends from [EventSourcedActor](http://rbmhtechnology.github.io/eventuate/user-guide.html#event-sourced-actors). One per user in the system. Created when a user signs up. Persists user events to the event log.

_TradeManager_ :
 Extends from [EventSourcedView](http://rbmhtechnology.github.io/eventuate/architecture.html#event-sourced-views). One per user in the system. Created by the UserActor when user signs up. Stores all trades for a particular user in memory. Routes all trade create and update message to respective trade actor.

_TradeActor_ :
Extends from [EventSourcedActor](http://rbmhtechnology.github.io/eventuate/user-guide.html#event-sourced-actors). One per trade. Created by trade manager when trade create command is received. Receives all trade create and update commands from the TradeManager and persists to the event log.

_TradeViewAggregateActor_ :
Extends from [EventSourcedView](http://rbmhtechnology.github.io/eventuate/architecture.html#event-sourced-views). One per system. Created on application startup. Receives all events from the event log handles only trade events and sends trade messages to connected Web Socket users.


### Whats Next

We will follow TDD and start implementing our application. 

Terms above which you didn't understand will start making sense.

This should also give you a good idea on how to test asynchronous apps written using akka.

Read part 2 here - [Persisting Events](http://kunalkanojia.github.io/blog/persisting-events)

### References

- [http://codebetter.com/gregyoung/2010/02/20/why-use-event-sourcing/](http://codebetter.com/gregyoung/2010/02/20/why-use-event-sourcing/)

- [http://martinfowler.com/eaaDev/EventSourcing.html](http://martinfowler.com/eaaDev/EventSourcing.html)

- [Event Sourcing - Greg Young, Youtube](https://www.youtube.com/watch?v=8JKjvY4etTY)

- [http://rbmhtechnology.github.io/eventuate/](http://rbmhtechnology.github.io/eventuate/)

- [https://www.playframework.com/](https://www.playframework.com/)

