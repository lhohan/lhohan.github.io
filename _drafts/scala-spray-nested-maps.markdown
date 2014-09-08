---
layout: post
title:  "spray-json conversion of nested Map's to JSON"
categories: scala runtime spray
---

In this post we will explain how to convert nested `Map`'s in Scala to JSON using the [spray-json](https://github.com/spray/spray-json) library.
This exercise is useful in itself as a basic example on how to make use of the default infrastructure spray-json provides but
it also works around an issue in the library where this out-of-the-box feature does not work.

At Mendix we developed a basic web client in Scala to start a Mendix application using JSON commands similar as to how m2ee tools works.

Such a JSON command typically looks like:

```javascript
{
  action : "action_name"
  params : {
    param1 : "value1"
    param2 : 2
    param3 : {
      param31 : "value31"
    }
  }
}
```

In which the `params` section is used to pass arguments to any potential action we support or may support in the future.
Meaning it can contain a nested structure as shown with param31.

In Scala code we represent this structure not surprisingly with a `Map`:

```scala
Map("param1" -> "value1", param2 -> 2, "param3" -> Map("param31" -> "value31"))
```

To marshall Scala objects to JSON we are using the light-weight [spray-json](https://github.com/spray/spray-json) library
which is quite neat and easy to use. It offers auto conversion of standard Scala types to JSON including collections.

To test this conversion we write following test case using ScalaTest and a more complicated Map structure:
{% highlight scala linenos%}
package webclient

import org.scalatest.FunSuite
import spray.json._

class JsonTest extends FunSuite {

  test("Map to JSON conversion") {
    val testMap = Map(
      "param1" -> "value1",
      "param2" -> 2,
      "param3" -> Map(
         "param4" -> "value4",
         "param5" -> Map(
            "param6" -> "value6",
            "param7" -> "value7"
         )
      )
    )

    assertResult( """{"param1":"value1","param2":"2","param3":{"param4":"value4","param5":{"param6":"value6","param7":"value7"}}}""") {
      testMap.toJson.toString()
    }
  }

}
{% endhighlight %}

Line 21: the expected result all in a one liner for convenience so we do not have to worry about new lines etc for verifying the result.

Line 22: `toJson` converts to an JSON AST (see [spray-json](https://github.com/spray/spray-json) doc which explains it very clearly) and calling toString gives us a plain string with the JSON output.

Running the test in `sbt` yields following error:

```
> test
[error] ....\JsonTest.scala:22:
Can not find JsonWriter or JsonFormat type class for scala.collection.immutable.Map[String,Any]
```

So the auto conversion of nested `Map`s does not seem to work. This is [currently an open issue](https://github.com/spray/spray-json/issues/33).

In order to make this work, here is how to implement our own 'protocol' that handles nested `Map`s:

{% highlight scala linenos%}
package webclient

import spray.json._

object M2eeJsonProtocol extends DefaultJsonProtocol {

  implicit object MapJsonFormat extends RootJsonFormat[Map[String, Any]] {
    def write(m: Map[String, Any]) = {
      JsObject(m.map {
        case (k, v) => v match {
          case v: String => (k, JsString(v))
          case v: Map[String, Any] => (k, write(v))
          case _ => (k, JsString(v.toString))
        }
      })
    }

    def read(value: JsValue) = ???
  }
}
{% endhighlight %}

To make this work implemented our own JSON converter using Spray infrastructure which is not that hard. To fix add :

The only thing we need to do is bring this implicit converter into scope by adding an import statement (line 5) to our test class:

{% highlight scala linenos%}
package webclient

import org.scalatest.FunSuite
import spray.json._
import M2eeJsonProtocol._

class JsonTest extends FunSuite {
...
{% endhighlight %}


Running our test again:

```
> test
[info] JsonTest:
[info] - Map to JSON conversion
[info] Run completed in 279 milliseconds.
[info] Total number of tests run: 1
[info] Suites: completed 1, aborted 0
[info] Tests: succeeded 1, failed 0, canceled 0, ignored 0, pending 0
[info] All tests passed.
```

Code project with failing test is [here] (https://github.com/lhohan/spray-json-pg/tree/d56776b0a1d5c96338065abe6dcd9b179c92c429).
Code project with the fix is [here] (https://github.com/lhohan/spray-json-pg/tree/4d7de8da8b456aa05f0068d26553412ed27baf21).

In case of questions of useful addition please to not hesitate to leave a comment.