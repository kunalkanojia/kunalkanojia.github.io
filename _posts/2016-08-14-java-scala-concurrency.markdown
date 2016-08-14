---
layout: page
title:  "Java & Scala concurrency comparison"
quote: Scala is easier than java come take a look. (More code snippets than fancy talk)
comments: true
---
 
> I was a happy java developer until I stated coding in scala.

In this post, I'll try to convince you to move to scala by showing you how much easier it is to write concurrent & async programs in scala than in java.

### Problem Statement - Word Counting

We will write a program to get files and word count in each file from a given directory.
The program will essentially do the following things.

1. Get all files in the directory
2. Count words in all the files
3. Return a Map of file name and total word count.

### Solution 1 - Java Serial Implementation

A very simple implementation of java code should look like this.

<script src="https://gist.github.com/kunalkanojia/4e1c940afcc9d7f1905ca77ca5b52770.js"></script>

Nothing fancy here, method `getWordCountForFiles(String path)` gets the job done sequentially for each file.

### Solution 2 - Java Concurrent Implementation 

Now after this gets rejected in the code review. We go back to our desk and use some java concurrency constructs so that the solution scales.
 
To do that in java we all know what to do - use the `Callable` and give it to our favourite `ExecutorService` and count the words for all files in parallel.

So the `getWordCountForFiles` method now creates a executor service, invokes our callable and then collects the results from each one of them.

Below code should be easy to understand and relate to for every java programmer.
[ I have tried to use java 8 features so as to keep the comparison as honest as possible.] 
 
<script src="https://gist.github.com/kunalkanojia/5541b0abe05c447f05116372d244a88c.js"></script>


As of last year I would be happy with the above code. It has minimal boiler plate, I use some of the fancy Java 8 features and now its doing the tasks concurrently so performance is also good.

But not today, Scala has spoilt me. I dont want to write so much code just to make it concurrent. 


### Solution 3 - Scala Futures FTW!

> The central drive behind Scala is to make life easier and more productive for the developer. -- Martin Odersky

Let me introduce you to scala `Future`

A Future is an object that can hold a value that may become available, as its name suggests, at a later time. It essentially acts as proxy to an actual value that does not yet exist.

To execute something async all I need to do is wrap it inside a Future.

{% highlight scala %}
//someFuture will hold the result of the computation and T represents the type of the result.
def someFuture[T]: Future[T] = Future {
  someExpensiveComputation()
}
{% endhighlight %}


Yes that is it, you can read more on how it works in detail [here](http://docs.scala-lang.org/overviews/core/futures.html)
 
Now lets see how we would implement this word count program in scala and how different it would look. 

<script src="https://gist.github.com/kunalkanojia/1e4f0295bc2666ba9621106d022ec36e.js"></script>

The above program is almost half the size of what our java program looks like and every method of it is async.

Method `getWordCountForFiles` uses for comprehension. For-comprehensions can be used to register new callbacks (to post new things to do) when the future is completed, i.e., when the computation is finished.
 
So it invokes `getFilesList` and once the result of `getFilesList` is available `processFiles` is invoke with the list of files. The result of `processFiles` is returned once it is available.

The methods `getWordCount` and `processFiles` are exactly similar to our java implementation. Just that they are wrapped in a Future which saves me from writing all the executor service code.

We just need to specify the ExecutionContext. ExecutionContext is an abstraction over a thread pool that is responsible for executing all the tasks submitted to it. Notice the `import ExecutionContext.Implicits.global`, we specify the default global execution context available in the Scala library.

Isn't this so much better than the Java implementation?

### Testing Async programs

If you are happy with the scala code, you must be wondering how to write test cases for such async code.
Don't worry the scala community has got you covered and its very easy. 

We use scala test library. It has got `whenReady` construct which can be used like below to test the program we just wrote.

{% highlight scala %}
class WordCountWithFutureSpec extends WordSpec with MustMatchers with ScalaFutures {

  "A word count helper" should {

    "return correct number of files and the word count" in {
      val resultFuture = WordCountWithFuture.getFileCount("src/main/resources/")
      
      whenReady(resultFuture){ result =>
        result.size mustEqual 2
        result.get("File1.txt") mustBe Some(6480000)
        result.get("File2.txt") mustBe Some(1000000)
      }
    }
  }
}
{% endhighlight %}

### Code

Both java and scala code with tests can be found here -

[Java Word Count](https://github.com/kunalkanojia/java_word_count)

[Scala Word Count](https://github.com/kunalkanojia/scala_word_count)
  
  
### Actor Model

> The Actor Model provides a higher level of abstraction for writing concurrent and distributed systems. It alleviates the developer from having to deal with explicit locking and thread management, making it easier to write correct concurrent and parallel systems.

In my next post we will implement the same programs with the actor model using Akka and see how much easier concurrency can get.
And we will do it in both java and scala.
  





