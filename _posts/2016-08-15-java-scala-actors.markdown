---
layout: page
title:  "Concurrency with Actor Model"
quote: Concurrency with Akka Actors using both scala and java.
comments: true
---
  
### Actor Model

> The Actor Model provides a higher level of abstraction for writing concurrent and distributed systems. It alleviates the developer from having to deal with explicit locking and thread management, making it easier to write correct concurrent and parallel systems.

Continuing from my last concurrency post we will try to solve the word counting problem using Actors. We will do the thing with java first and then the same thing with scala. 

### Problem Statement - Word Counting

We will write a program to get files and word count in each file from a given directory.
The program will essentially do the following things.

1. Get all files in the directory
2. Count words in all the files
3. Return a Map of file name and total word count.

### Java Implementation #TODO [Use the new Java8 API]

- Test

<script src="https://gist.github.com/kunalkanojia/8202d0690208949054eee8b88cec8da2.js"></script>

- Implementation

<script src="https://gist.github.com/kunalkanojia/a540eef7eafe00dd253f18e9351367f6.js"></script>

### Scala Implementation

 - Test
 
 <script src="https://gist.github.com/kunalkanojia/918becb053956ad24ceba24bb12f3134.js"></script>

 - Implementaion

<script src="https://gist.github.com/kunalkanojia/36bacddf2b4a947e215580b68c660878.js"></script>
