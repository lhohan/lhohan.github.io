---
layout: post
title:  "Conversion of nested Maps using spray-json"
category: scala
categories: scala runtime spray
author: hans_l'hoest
---

A Mendix application can be started by only sending JSON actions to a running a Mendix Runtime. 
Recently we developed a web client in Scala to start a Mendix application using only JSON commands similar as to how m2ee tools works. 
In this post we will explain how to convert nested `Map`s in Scala to JSON using the [spray-json](https://github.com/spray/spray-json) library.

It is a basic example to show how easy it is the convert your own types into JSON using the default infrastructure provided by spray-json and
at the same time works around an issue in the library where these kind of `Map`s are not supported.

A JSON command sent to the server (the Mendix Runtime) to execute an action typically looks like:

```javascript
{
  action : "action_name"
  params : {
    param1 : "value1"
    param2 : 2
    param3 : {
      param4 : "value4"
    }
  }
}
```

The `params` section is used to pass arguments to any potential action we support or may support in the future.
Meaning it can contain a hierarchical structure as shown.

In Scala code we represent this structure not surprisingly with a `Map`:

```scala
Map("param1" -> "value1", param2 -> 2, "param3" -> Map("param4" -> "value4"))
```

To marshall Scala objects to JSON we are using a neat, easy to use, light-weight library called [spray-json](https://github.com/spray/spray-json).
It offers auto conversion of standard Scala types to JSON including collections.

To test the conversion we need, we write following test case using ScalaTest and a more complicated `Map` structure:

```scala
import org.scalatest.FunSuite
import spray.json._
import M2eeJsonProtocol._

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

    assertResult( """{"param1":"value1","param2":"2","param3":{"param4":"value4","param5":{"param6":"value6","param7":"value7"}}}""") {  // 1
      testMap.toJson.toString()  // 2
    }
  }
}

```

(1): the expected result all in a one liner for convenience so we do not have to worry about new lines etc. for verifying the result.

(2): `toJson` converts to an JSON AST (see [spray-json](https://github.com/spray/spray-json) doc which explains this very clearly) and calling toString gives us a plain string with the JSON output.

Running the test in `sbt` yields following error:

```
> test
[error] ....\JsonTest.scala:...:
Can not find JsonWriter or JsonFormat type class for scala.collection.immutable.Map[String,Any]
```

So conversion of nested `Map`s does not work out of the box and it is [currently an open issue](https://github.com/spray/spray-json/issues/33).

In order to make this work, here is how to implement our own [Json protocol](https://github.com/spray/spray-json#jsonprotocol) that handles nested `Map`s:

```scala
import spray.json._

object M2eeJsonProtocol extends DefaultJsonProtocol {

  implicit object MapJsonFormat extends JsonFormat[Map[String, Any]] { // 1
    def write(m: Map[String, Any]) = {
      JsObject(m.map {                                                 // 2
        case (k, v) => v match {
          case v: String => (k, JsString(v))
          case v: Map[String, Any] => (k, write(v))                    // 3
          case _ => (k, JsString(v.toString))                          // 4
        }
      })
    }

    def read(value: JsValue) = ???                                     // 5
  }
}
```

We implement a [JsonFormat](https://github.com/spray/spray-json#jsonprotocol) which is part of the default infrastructure `spray-json` provides to support custom conversions. 
We extend JsonFormat parametrized with the type we want to convert, in our case `Map[String, Any]` (1). 
We return a `JsObject` (2) initialized with the result of mapping each element (2) to a key value pair for which we convert every value. 
If this value is of type `Map[String, Any]` (3) we make a recursive call, this the main construct that makes our conversion work. 
Otherwise we simply convert to a string (4), this is all we need for now, in case of numbers or other structures this implementation can be relatively easy extended.

The read part we do not need, so we leave it unimplemented (5).     

The only thing left to do is bring this implicit converter into scope by adding an import statement (1) to our test class:

```scala
import org.scalatest.FunSuite
import spray.json._
import M2eeJsonProtocol._  // 1

class JsonTest extends FunSuite {
...
```


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

Code project with the fix is [here] (https://github.com/lhohan/spray-json-pg/tree/65d4f4b4b84af9577cad213e7c3e0a1f0377b1ef).

In case of questions or useful additions please to not hesitate to leave a comment.
