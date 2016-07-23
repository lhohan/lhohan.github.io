---
layout: post
title:  "FizzBuzz Stream fun"
categories: scala
---

*Updated with a more functional implementation of FizzBuzz November 2015*
*Updated with a link to an implementation using `Monoids` July 2016*

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

As a little extra, this is how we could test our FizzBuzz implementation using [ScalaCheck](http://www.scalacheck.org/):

{% gist lhohan/9039481 %}

*A more functional implementation of FizzBuzz (added November 2015)*

Thanks to [Dierk König's implementation in Frege][frege]:

{% gist lhohan/4eab1c83f7e9e17216b3 %}

What I like about this implementation: 

- Much less conditionals. There is only 1 compared to 4 in the first implementation. And the order matters in the latter, if I do not put the `i % 15 == 0` check first the logic will fall apart. So if changes need to be made it is easier to introduce bugs in the one containing more conditionals.
- The code can be more easily built incrementally using the REPL. It just *flows* nicer. The `shout` method needs to be entered in one go, all or nothing.
- Closer to the specification. Consider the `i % 15 == 0` check in the first implementation a bit more: to fit the FizzBuzz specification better it would be more accurate to write it as  `i % 3 == 0 ||  i % 5 == 0`. This would up the conditional checks to 5 though. At first sight the concatenation in the functional implementation felt like a lucky shot, but in fact it fits exactly the specifaction of FizzBuzz: I suppose it is no coincidence we are shouting 'FizzBuzz', a concatentation, instead of 'Boo!'.

Note there is also `zipWithIndex` in Scala but I maintained the same "`zip` and `map`"-logic to keep the code more consistent and hence more simple.

*[A Fizzbuzz implementation using `Monoids` (Added July 2016)](monoid_example)* When time permits
I may expand this post with this example.


[frege]: https://dierk.gitbooks.io/fregegoodness/content/src/docs/asciidoc/fizzbuzz.html
[monoid_example]: https://www.reddit.com/r/scala/comments/45gqpd/whats_a_monoid/czy732k

