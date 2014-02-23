---
layout: post
title:  "FizzBuzz Stream fun"
categories: scala
---

[FizzBuzz](http://en.wikipedia.org/wiki/Fizz_buzz) fun in Scala using Scala
[Streams](http://www.scala-lang.org/api/current/index.html#scala.collection.immutable.Stream).

The straightforward implementation of FizzBuzz usually involves defining a list or array of fixed size 100, populate it
using a loop over the index and set the value in the list according to our Fizz Buzz specification.

[Streams](http://www.scala-lang.org/api/current/index.html#scala.collection.immutable.Stream) are lazy lists in which elements are only evaluated when they are needed. This allows for a seemingly
infinite recursive loop looking definition as shown in the Fibonacci example below. The first line builds a sequence
of sums, the sum of `a` and `fibFrom` looks as it if will never end if not for the lazy evaluation.

{% gist /lhohan/9038316 %}

The 2nd line is there for clarity to show how to calculate the nth Fibonacci number based on our `Stream` definition. The
`take` method will return a Stream still, so nothing is being evaluated until `toList` is called. (Note that in case
of an infinite collection `toList` may never terminate.)

Here is an alternate implementation of FizzBuzz using the [Stream](http://www.scala-lang.org/api/current/index.html#scala.collection.immutable.Stream) class in the standard Scala API.

{% gist lhohan/854a2c516d016ac22236 %}

[`Stream.from(start:Int)`](http://www.scala-lang.org/api/current/index.html#scala.collection.immutable.Stream$) creates
and infinite stream of integers and we map them using a function according to the FizzBuzz specification. We then take
the first `n` of this infinite list and convert it to a 'normal' Scala List.

To print the first 100:
{% highlight scala lineno %}
  fizzbuzz(100).foreach{ println(_) }
{% endhighlight %}

As a little extra, this how we could test our FizzBuzz implementation using [ScalaCheck](http://www.scalacheck.org/):

{% gist lhohan/9039481 %}


