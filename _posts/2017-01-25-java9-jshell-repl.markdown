---
layout: page
title:  "jshell - The java REPL "
quote: Checkout features in the new java REPL(read eval print loop).
comments: true
---

When I had started with Scala the first thing that I loved was the Scala REPL(read–eval–print loop).
It’s a great tool to learn Scala and test code snippets.

While back in the java world we had to write a class with main method to try something. But with Java 9 release this will change.
JShell will be our shiny new java REPL - [http://openjdk.java.net/jeps/222](http://openjdk.java.net/jeps/222) .


To get started with java 9 currently you can download and install the early access version from here - [https://jdk9.java.net/download/](https://jdk9.java.net/download/).

### Starting the REPL

Jshell is part of the jdk bin directory so if u have that in your path you can just run the jshell command to start the REPL.

{% highlight bash %}
bash-3.2$ jshell
|  Welcome to JShell -- Version 9-ea
|  For an introduction type: /help intro

jshell>
{% endhighlight %}


There is a verbose mode which is feel will be very helpful for beginners. To start in verbose mode you can run : jshell –v.

To enter or exit verbose mode once you are in the repl you can use –
/set feedback verbose and /set feedback normal.


{% highlight bash %}
jshell> /set feedback verbose
|  Feedback mode: verbose

jshell> String a = "A"
a ==> "A"
|  created variable a : String

jshell> /set feedback normal
|  Feedback mode: normal

jshell> String b = "B"
b ==> "B"
{% endhighlight %}

The verbose mode will be a good way to learn java.

### Expressions
You can write any valid java expression and jshell will execute it and assign it to a variable.

{% highlight bash %}
jshell> 3 * 3
$1 ==> 9
|  created scratch variable $1 : int

jshell> $1
$1 ==> 9
|  value of $1 : int
{% endhighlight %}


### Variables
Variables can be defined just like they are defined in a java program. Once a variable is defined it is present in the scope.

{% highlight bash %}
jshell> String str = "Hello World!"
str ==> "Hello World!"
|  created variable str : String

jshell> str
str ==> "Hello World!"
|  value of str : String
{% endhighlight %}

> Did u notice no semicolons!

### Methods

{% highlight bash %}
jshell> void printHelloWorld(){
   ...> System.out.println("Hello World!");
   ...> }
|  created method printHelloWorld()

jshell> printHelloWorld()
Hello World!
{% endhighlight %}

### Class

{% highlight bash %}
jshell> class SampleClass {
   ...> public static void hello(){
   ...> System.out.println("hello");
   ...> }
   ...> }
|  created class SampleClass

jshell> SampleClass.hello()
hello
{% endhighlight %}


### Commands
Jshell provides some helpful commands for you to get more information. Type /help and you will know all the list of commands. Here I have it for you to browse -

{% include image.html url="/images/jshell/help_cmd.png" style="text-align: center;" %}


### Networking
Jshell is not just for testing local code snippets. It gives you networking access which opens all sorts of possibilities.

{% highlight bash %}
jshell> URL url = new URL("http://kkanojia.me")
url ==> http://kkanojia.me
|  created variable url : URL

jshell> url.openConnection().getContentType()
$4 ==> "text/html; charset=utf-8"
|  created scratch variable $4 : String
{% endhighlight %}

### Exceptions
One of my favourite things about jshell is it wraps checked exceptions in the background.

Here I read and print a file without catching IOEXception.

{% highlight bash %}
jshell> Path path = Paths.get("/Users/kunalkanojia/file.txt")
path ==> /Users/kunalkanojia/file.txt
|  created variable path : Path

jshell> Stream<String> lines = Files.lines(path)
lines ==> java.util.stream.ReferencePipeline$Head@78e67e0a
|  created variable lines : Stream<String>

jshell> lines.forEach(System.out::println)
this is sample
{% endhighlight %}


### Forward reference
Another good thing is you dont have to worry about defining your methods and variables in correct order.
The forward reference support is pretty good.

{% highlight bash %}
jshell> double area(double radius){
   ...> return PI * square(radius);
   ...> }
|  created method area(double), however, it cannot be invoked until variable PI, and method square(double) are declared
{% endhighlight %}


### JavaDOC

Pressing shift tab will get you the javadoc

{% highlight bash %}
jshell> java.nio.file.Paths
java.nio.file.Paths
<press shift-tab again to see javadoc>

jshell> java.nio.file.Paths
java.nio.file.Paths
This class consists exclusively of static methods that return a Path by
converting a path string or URI .
{% endhighlight %}

### Conclusion
The repl is definitely something which I think people will find useful.

Some features I would love to see:

  1. Paste Mode: to copy multiline commands
  2. Syntax Highlighting: Who doesn’t want that
  3. Multiline editing.

Having said that, I really like jshell.
The tab auto-completion is quite good and  features like forward reference & exception handling really make it usable.
The ability to open external editor using /edit is cool.

> It’s time to get rid of that TestMain.java class we all have in our workspace.
