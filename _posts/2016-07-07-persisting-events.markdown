---
layout: page
title:  "Event Sourcing with scala, play framework and eventuate (Part 2 - Persisting Actor States)"
quote:   This is the basic idea behind event sourcing -  instead of storing current application state, the full history of changes is stored as immutable facts and current state is derived from these facts.
comments: true
---

In this section we will complete two tasks

1. Writing our persistent actors to persist trade events to LevelDB. (C of CQRS)

2. Reading events from a shared event log and maintain state. (Q of CQRS)

For this we will be using two abstractions provided by eventuate - Event-sourced Actor and Event-sourced View.

## Event-sourced Actor

An event-sourced actor is an actor that captures changes to its internal state as a sequence of events. It persists these events to an event log and replays them to recover internal state after a crash or a planned re-start. Its derived state is an in-memory write model, representing the command-side (C) of CQRS.

## Trade Actor: Persisting events

Lets start by writing a test case for TradeActor.

We want our trade actor to be able to persist events when it receives a `CreateTrade` & `UpdateTrade` commands.

The Spec class skeleton looks like this - 

<script src="https://gist.github.com/kunalkanojia/1a737882d4f3102e907f6f2d8a6eaf0c.js"></script>

We are using [akka-testkit](http://doc.akka.io/docs/akka/current/scala/testing.html) for testing our actor interactions.

The test case creates a unique event log for itself so that other tests running in parallel wont affect this testcase. 

We will be doing three things in the test,

 - Create a trade actor
 - Send command messages
 - Assert that we got the correct response

The complete test case looks like below - 

<script src="https://gist.github.com/kunalkanojia/8df990962974c1577ce48f3ee1962fc7.js"></script>

Simple enough! One thing missing here is that we are not clearing the event log before each test run. I am leaving it out as a TODO but if you want to do it, you can write before method which deletes the leveldb directory.

Now lets start implementing the trade actor.

Our trade actor class will extend from `EventSourcedActor`. 
When implementing a EventSourcedActor we need to override two methods `onCommand` and `onEvent`.
We will maintain out current state in `tradeOpt` variable.

There are three steps in Trade Actor when creating or updating a trade, 
 
 - `onCommand` handler will handle the create/update command and call `persist` with  the respective event.

 - The `onEvent` handler is invoked on successful persist. This is where we will update our internal state.

 - After completion of `onEvent` the persist method calls the handler where we return success or failure message to the sender.
 
<script src="https://gist.github.com/kunalkanojia/2da73648265f7697fe2af0d477b5e48d.js"></script>

That's it. We are able to persist changes and maintain current state of our trade object now.

On restart the `onEvent` handler is invoked in the same sequence as the events were received. So we will always have the same state even when the application restarts or actor crashes.


## Event-sourced View
Event sourced view is an actor that consumes events from its event log but cannot produce new events. Its derived state is an in-memory read model, representing the query-side (Q) of CQRS.


## Trade Manager : Query side

Our Trade manager will extend from Event-sourced view.

It will route Create and Update commands to respective trade actors based on trade id. It will also receive `TradeCreated` and `TradeUpdated` events asynchronously from the Trade Actor on successful persist to levelDB. On receiving these events our manager will update the internal state so that all queries can be implemented in trade manager.
 
The spec class for Trade Manager will be very similar to our trade Actor. 

Lets go ahead and look at the completed spec.

<script src="https://gist.github.com/kunalkanojia/fbecdc1cd7caf1b349f5dba0c1051d49.js"></script>

Again here we follow same principle, send certain commands to our trade manager actor and expect correct responses.

So our Trade Manager actor will have to handle those four commands we mentioned in the spec and  also keep in memory the list of trades received in a Map for retrieving individual or all trades.

<script src="https://gist.github.com/kunalkanojia/a5531a1f09a362ea7b82876a168c2769.js"></script>

The trade manager above is complete but it does not receive the `onEvent` calls from the `TradeActor`. Now we need the TradeActor to asynchronously send message to the manager on successful persist.

Eventuate has got that covered for us. 

## Event Routing 
An event that is emitted by an event-sourced actor or processor can be routed to other event-sourced components if they share an Event log.

Eventuate defines the following routing rules: 

  - If an event-sourced component has an undefined aggregateId, all events are routed to it. It may choose to handle only a subset of them though.
  
 - If an event-sourced component has a defined aggregateId, only events emitted by event-sourced actors or processors with the same aggregateId are routed to it.


In eventuate the routing destinations are defined during emission of an event and are persisted together with the event. This makes routing decisions repeatable during event replay and allows for routing rule changes without affecting past routing decisions.

If you look at the definition of `persist` method its this - 

{% highlight scala %}

final def persist[A](event: A, customDestinationAggregateIds: Set[String] = Set())(handler: Handler[A]): Unit
      
{% endhighlight %}

So we will pass managers aggregate Id to the persist method in TradeActor and our trade manager will start receiving the create and update events.

{% highlight scala %}

class TradeActor(){ 
 //...Notice the managers aggregate id
case CreateTrade(trade) =>
      persist(TradeCreated(trade), Set(managerAggregateId)) {
        case Success(evt) =>
          sender() ! CreateTradeSuccess(trade)
        case Failure(cause) =>
          sender() ! CreateTradeFailure(cause)
      }
{% endhighlight %}


All tests should pass after you have made the above change to the trade actor.

That's it, with eventuate we have easily implemented the command and the query for our application.

## Reference

[http://rbmhtechnology.github.io/eventuate/reference.html](http://rbmhtechnology.github.io/eventuate/reference.html)
