---
layout: page
title:  "Concurrency with Akka Actors"
quote: Concurrency with Akka Actors using Scala, Java and Kotlin.
comments: true
---
  
### Actor Model

> The Actor Model provides a higher level of abstraction for writing concurrent and distributed systems. It alleviates the developer from having to deal with explicit locking and thread management, making it easier to write correct concurrent and parallel systems.

Continuing from my last concurrency post we will try to solve the word counting problem using Actors. We will do the thing with java first, then the same thing with scala and bonus one with Kotlin.

### Problem Statement - Word Counting

We will write a program to get files and word count in each file from a given directory.
The program will essentially do the following things.

1. Get all files in the directory
2. Count words in all the files
3. Return a Map of file name and total word count.

### Solution Strategy

We will be using [Akka](http://akka.io/).

Our solution is going to be simple. We will have two actors `WordCountMaster` and `WordCountWorker`.

<b>WordCountMaster</b> : 

It does two tasks creates worker actors and collates results from the workers.

It accepts the following messages :

<em>StartCounting message: </em> Master actor receives this message and starts the task. It reads all files from the directory, creates the worker actors and tells each worker to start counting the words in each file.

<em>WordCount message: </em> On receipt of WordCount message from each worker, the master collates the result and responds with the result to the  original sender.

<b>WordCountWorker</b> : 

Works on individual files to count the number of words in the file. Starts the task on receiving <em>FileToCount</em> message.


### Java Implementation

- Test

<script src="https://gist.github.com/kunalkanojia/8202d0690208949054eee8b88cec8da2.js"></script>

- Implementation

<script src="https://gist.github.com/kunalkanojia/a540eef7eafe00dd253f18e9351367f6.js"></script>

### Scala Implementation

 - Test
 
 <script src="https://gist.github.com/kunalkanojia/918becb053956ad24ceba24bb12f3134.js"></script>

 - Implementation

<script src="https://gist.github.com/kunalkanojia/36bacddf2b4a947e215580b68c660878.js"></script>


### Kotlin Implementation

Kotlin is a new programming language from JetBrains.

It fixes a lot of issues we have with java - [https://kotlinlang.org/docs/reference/comparison-to-java.html](https://kotlinlang.org/docs/reference/comparison-to-java.html)

And I also meet a lot of people who are optimistic about kotlin adoption. 

So I gave it a try and this is how the code looks. Definitely better than java, but Scala still stays my favourite.

<script src="https://gist.github.com/kunalkanojia/198f6c063bd12621341827330e59171c.js"></script>
