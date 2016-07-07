---
layout: page
title:  "Event Sourcing with scala, play framework and eventuate (Part 2 - Persisting Actor States)"
quote:   This is the basic idea behind event sourcing -  instead of storing current application state, the full history of changes is stored as immutable facts and current state is derived from these facts.
comments: true
---

In this section we will do two tasks 

1. Write our persistent actors to persist trade events to LevelDB.

2. Read events from a shared event log and maintain state.

For this we will be using two abstractions provided by eventuate - Event-sourced Actor and Event-sourced View.

## Event-sourced Actor

An event-sourced actor is an actor that captures changes to its internal state as a sequence of events. It persists these events to an event log and replays them to recover internal state after a crash or a planned re-start. Its derived state is an in-memory write model, representing the command-side (C) of CQRS.

## Trade Actor: Persisting events

Lets start by writing a test case for TradeActor.

We want out trade actor to be able to persist events when it receives a `CreateTrade` message `UpdateTrade` commands.
The Spec class skeleton looks like this - 

<script src="https://gist.github.com/kunalkanojia/1a737882d4f3102e907f6f2d8a6eaf0c.js"></script>

We are using (akka-testkit)[http://doc.akka.io/docs/akka/current/scala/testing.html].

The test case creates a unique event log for itself so that other tests running in parallel wont affect this testcase. 

We will be doing three things in the test,

 - Create a trade actor
 - Send message
 - Assert that we got the correct response

The complete test case looks like below - 

<script src="https://gist.github.com/kunalkanojia/8df990962974c1577ce48f3ee1962fc7.js"></script>

Simple enough! One thing missing here is that we are not clearing the event log before each test run.

Now lets start implementing the trade actor.

Our trade actor class will extend from `EventSourcedActor`. 
When implementing a EventSourcedActor we need to override two methods `onCommand` and `onEvent`.
`tradeOpt` variable is where we will maintain the current state of the event

There are three steps in Trade Actor when creating or updating a trade, 
 
 - `onCommand` handler will handle the create/update command and call `persist` with `TradeCreated`.

 - The `onEvent` handler is invoked on successful persist. This is where we will update our internal state.

 - After completion of `onEvent` the persist method calls the handler where we return success or failure message to the sender.
 
<script src="https://gist.github.com/kunalkanojia/2da73648265f7697fe2af0d477b5e48d.js"></script>

That's it we are able to persist changes and maintain current state of our trade object now.

On restart the `onEvent` handler is invoked in the same sequence as the events were received. So we will always have the same state even when the application restarts.


## Event-sourced View
Event sourced view is an actor that consumes events from its event log but cannot produce new events. Its derived state is an in-memory read model, representing the query-side (Q) of CQRS.


## Trade Manager : Query side

Our Trade Manager will route Create and Update commands to respective trade actors. It will also receive `TradeCreated` and `TradeUpdated` events asynchronously from the Trade Actor to update its state.
 
 
The spec class for Trade Manager will be similar to our trade Actor. 

Lets go ahead and look at the completed spec.


<script src="https://gist.github.com/kunalkanojia/fbecdc1cd7caf1b349f5dba0c1051d49.js"></script>


//TODO - Complete Trade manager and event collaboration between actor and manager. 