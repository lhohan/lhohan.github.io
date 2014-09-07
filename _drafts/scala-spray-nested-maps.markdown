---
layout: post
title:  "spray-json conversion for type Map"
categories: scala runtime spray
---

When a runtime container is started a deployed Mendix application can be started by sending it HTTP JSON requests.

In order un/marshall Scala objects to and from JSON we are using the [spray-json](https://github.com/spray/spray-json) library.

Basically what we want is a Scala case class like:


to be converted to JSON like:


The TODO BLAH part may any arbitrary JSON.
So we need to convert a (potentially nested) structure of ´Map´s to JSON.

Spray offers auto conversion of standard Scala types to JSON including collections.

To test this conversion we write following test case using ScalaTest:
{% highlight scala linenos%}
package com.mendix.util.webclient

import org.scalatest.FunSuite
import spray.json._

class JsonTest extends FunSuite {

  test("Map to Json conversion") {
    val testMap = Map("A" -> "aaa", "B" -> "bbbb", "X" -> Map("C" -> "cccc", "Y" -> Map("D" -> 4, "E" -> "lalaeee")))
    assertResult( """{"A":"aaa","B":"bbbb","X":{"C":"cccc","Y":{"D":"4","E":"lalaeee"}}}""") {
      testMap.toJson.toString()
    }
  }

}
{% endhighlight %}

Running the test yields following error:
{% highlight scala %}
[error] ...
Can not find JsonWriter or JsonFormat type class for scala.collection.immutable.Map[String,Object]
{% endhighlight %}

So the built-in auto conversion of ´Map´s does not seem to work. Checking spray-json  See also
following Spray issue [TODO insert link]

While building a web client in Scala using Spray to send HTTP commands to the Mendix runtime we ran into an issue when handling nested Maps.

Consider:

Code project with failing test is here [TODO insert link to branch].
Code project with the fix is here [TODO insert link to branch].


To make this work implemented our own JSON convertor using Spray infrastructure which is not that hard. To fix add :

Et voila:
