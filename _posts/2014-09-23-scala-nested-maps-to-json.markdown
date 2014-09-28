---
layout: post
title:  "Conversion of nested Maps to JSON in Scala using spray-json"
category: scala
categories: scala runtime spray
author: hans_l'hoest
---

Recently we at Mendix developed a web client in Scala to start a Mendix application using only JSON commands similar as to how [m2ee-tools](https://github.com/mendix/m2ee-tools/) works.

While doing so we needed a way to convert nested Scala `Map`'s into JSON generically. 
In this post I will explain how we implemented this using the [spray-json](https://github.com/spray/spray-json) library.

It is a basic example to show how easy it is the convert your own types to JSON using the default infrastructure provided by [spray-json](https://github.com/spray/spray-json) and
at the same time we will work around an issue in the library where nested `Map`s are not supported. The source code is also included 
as [a fully functional `sbt` project](https://github.com/lhohan/spray-json-pg) as well.

To set the stage: a typical JSON command sent to the server (the Mendix Runtime) to execute an action typically looks like:

```javascript
{
  "action": "update_appcontainer_configuration",
  "params": {
    "runtime_port": "9080",
    "runtime_jetty_options": {
      "set_stats_on": "true"
    }
  }
}
```

The `params` section is used to pass arguments for any potential `action` we support or may support in the future.
Meaning it can contain a hierarchical structure.

In Scala code we may represent such a structure not surprisingly with a `Map`:

```scala
Map(
    "action" -> "update_appcontainer_configuration", 
    "params" -> Map(
      "runtime_port" -> "9080",
      "runtime_jetty_options" -> Map(
        "set_stats_on": "true"
      )
    )
)
```

To marshall Scala objects to JSON we use [spray-json](https://github.com/spray/spray-json).
The documentation of this library is pretty solid so there’s no need to go over the basics really. The feature we go into a bit deeper here is its
 (auto) conversion of standard Scala types to JSON including collections.

Test first
-------

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

    assertResult( """{"param1":"value1","param2":2,"param3":{"param4":
  "value4","param5":{"param6":"value6","param7":"value7"}}}""") {  // 1
      testMap.toJson.toString()  // 2
    }
  }
}
```

(1): the expected result all in a one liner for convenience so we do not have to worry about new lines etc. when verifying the result.

(2): `toJson` converts to an JSON AST and calling toString gives us a plain string with the JSON output.

Running the test in `sbt` yields following error:

```
> test
[error] ....\JsonTest.scala:...:
Can not find JsonWriter or JsonFormat type class for scala.collection.immutable.Map[String,Any]
```

So conversion of nested `Map`s does not work out of the box and it is [currently an open issue](https://github.com/spray/spray-json/issues/33).

Implementation
------

In order to make this work, here is how to implement our own [Json protocol](https://github.com/spray/spray-json#jsonprotocol) that handles nested `Map`s:

```scala
import spray.json._

object M2eeJsonProtocol extends DefaultJsonProtocol {

  implicit object MapJsonFormat extends JsonFormat[Map[String, Any]] { // 1
    def write(m: Map[String, Any]) = {
      JsObject(m.mapValues {                  // 2
        case v: String => JsString(v)         // 3
        case v: Int => JsNumber(v)
        case v: Map[String, Any] => write(v)  // 4
        case v: Any => JsString(v.toString)   // 5
      })
    }

    def read(value: JsValue) = ???            // 6
  }
}
```

We implement a [JsonFormat](https://github.com/spray/spray-json#jsonprotocol) which is part of the default infrastructure `spray-json` provides to support custom conversions. 
We extend JsonFormat parametrized with the type we want to convert, in our case `Map[String, Any]` (1). 
We return a `JsObject` initialized with the result of mapping each element (2) to its appropriate `JsValue` object.
As you can see, we explicitly convert `String` and `Int` (3).  
Type `Map[String, Any]` (4) is handled recursively: this the main construct which makes our conversion work. 
All other types we simply convert to a string (5); a further improvement could be to convert *any* type (say by using `toJson`), 
but this suffices to make our test pass and JSON requests work in practice.

The `read` method, converting JSON to Scala `Map`s we do not need, so we leave it unimplemented (6).     

The only thing left to do is to bring this implicit converter into scope by adding an import statement (1) to our test class:

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

Code project with failing test is [here] (https://github.com/lhohan/spray-json-pg/tree/e2cda87713d6f835d756acafe9fc8f247cb23b4b) and the one containing the fix is [here] (https://github.com/lhohan/spray-json-pg/tree/84dd18227e8a6c488a502e8dbab0d0acfc0cddfd).

This concludes the main part of this post on converting our Scala classes into JSON. 


Getting rid of the compilation warning
---------

Off the main topic of this post but you probably noticed the warning: 

```scala
[warn] .../src/main/scala/M2eeJsonProtocol.scala:10: non-variable type argument 
String in type pattern Map[String,Any] is unchecked since it is eliminated by 
erasure
[warn]           case v: Map[String, Any] => (k, write(v))                    // 4
[warn]                   ^
[warn] one warning found
```

As Scala runs on the jvm, types are erased at runtime and the case pattern at (4) will not only match the `Map[String, Any]` type but *any* `Map` type.
There are various ways in which we could try to remove this warning but things may get more complicated pretty fast depending on the route chosen (interesting but non trivial path might be [type tags](http://docs.scala-lang.org/overviews/reflection/typetags-manifests.html)). However in our
case we do not have to go through great lengths to prevent this as other typed `Map`'s would generate invalid JSON anyway 
(a named element in JSON, the keys in our `Map`s, must be a string) and things will crash unexpectedly further down the line anyway.
One straightforward way to avoid unexpected errors would be to validate the input `Map` before marshalling but we focus here on removing the warning.  

In Scala 2.10 we can the following:

```scala
import spray.json._

object M2eeJsonProtocol extends DefaultJsonProtocol {

  type StringToAny = Map[String, Any]

  implicit object MapJsonFormat extends JsonFormat[Map[String, Any]] { // 1
    def write(m: Map[String, Any]) = {
      JsObject(m.mapValues {                  // 2
        case v: String => JsString(v)         // 3
        case v: Int => JsNumber(v)
        case v: StringToAny => write(v)  // 4
        case v: Any => JsString(v.toString)   // 5
      })
    }

    def read(value: JsValue) = ???            // 6
  }
}

```

This removes the warning but in Scala 2.11 the compiler figures out we only shifted the issue elsewhere:

```scala
[warn] .../src/main/scala/M2eeJsonProtocol.scala:12: non-variable type argument 
String in type pattern  scala.collection.immutable.Map[String,Any] (the 
underlying of M2eeJsonProtocol.StringToAny) is unchecked since it is eliminated 
by erasure
[warn]         case v: StringToAny => write(v)  // 4
[warn]                 ^
[warn] one warning found
```

A less elegant but, nevertheless, pretty clear and effective solution then would be to replace (4) with:

```scala
case v: Map[_, _] => write(v.asInstanceOf[Map[String, Any]])  // 4
```

This removes the warning in both Scala 2.10 and 2.11. 

The project without the warning is [here](https://github.com/lhohan/spray-json-pg/tree/bd518fd7e0217ac3b5473aa4b016083826b744ff). 

In case of questions or suggestions please do not hesitate to leave a comment.
