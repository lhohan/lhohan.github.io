
Below are the notes of Scala Days 2014.

Get a general feel of what is happening arround Scala, check out some things I do not know yet, get a lot of new impression and Reactive.

Before getting in what may turn out to be a TL;DR kind of post here is a telegram style of the highlights:
  - Very well organised and those 3 days where over before in a ... 18 sessions slots (x 4 tracks) varrying over a lot of different topics, every evening there was a community event (party at roof terrass of 15 story building anyone?), nice atmosphere, good food, ... . If you are interested in Scala and what's happening around it, this conference is the place to be.
  - If we had to pick only one session to watch I would go for the Reactive Streams talk by R. Kuhn and V. Klang. Of course your interest in the topic may be different than ours but is was a clear and well-strucutred talk on what Reactive Streams are including a demo.


A conference like Scala Days makes you go home with a head full of fresh ideas and feel 'enthused' on all the stuff you saw and learned.

All session where record by Parleys and will be made available [half of July] but a collection has already been started with a link to all slides and code:
[https://gist.github.com/kevinwright/9505828a0dcc0c4c0d56]()

One side effect is that such a conference is in fact only the beginning for a lot the things you discovered. Usually it is too much to follow up on but I'll add the things I have marked anyway.

The Simple Parts (Martin Odersky)
====

In this session the goals was to show that in essence Scala is simple and a response to some controversies living in the different communities around Scala.

It ranges over simple tips like  *'don't pack too much in one expression and find meaningful names'* to using implicits as a way of doing dependency injection.

Scala is unifier (OO & FP, scripting + solid language used in large projects to build their core business,...)

10 years old, we look at the original definition: "Scala is a scalable language" and a nice reference to James Iry's blog which you should really read if you haven't yet as it is hilarious.
Prof. Odersky's improved definition would be "Scala is modular language".

Below are the 7 fundamental actions that characterizes Scala to Prof. Odersky, the other parts are not considered part of the core of Scala:

  - Compose
    - Everything is an expression
  - Decomposition
    - Pattern matching
  - Group
    - Everything can be grouped and nested in Scala
  - Recurse
    - almost always better than a loop
    - @tailrec annotated functions are guaranteed to be efficient
  - Abstract
  - Aggregate (collections)
    - uniformity of use
    - transform instead of CRUD
    - controversy around typing (the need for [use case] in Scaladoc to 'hide' the complexity of it all. Why not a functor? Because than arrays wouldn't work anymore. Hence the CanBuildFrom.)
  - Mutate
    - use with restraint but can cut down on the boilerplate and improve clarity
    - where it is used in [dotty](https://github.com/lampepfl/dotty)

The last part had some interesting slides picked form a larger presentation but some parts didn't hit home.
[http://functional.tv/post/86657991174/sf-scala-martin-odersky-scala-the-simple-parts](This talk given before at Workday) or [http://www.slideshare.net/Odersky/scala-the-simple-parts](this slide deck).



Aha moments or interesting things I never realized or thought about before:
"weak composition shortcomings in a language require external dependency mgt frameworks"

Thing to dig deeper in:

  - 'Why functional programming matters' by John Hughes
  -

Quotes:

Did not get, must look further:
   - mapValues question: surprise, ack'ed by Odersky.

