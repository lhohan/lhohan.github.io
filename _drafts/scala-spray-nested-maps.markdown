---
layout: post
title:  "Conversion of nested Maps using spray-json"
category: scala
categories: scala runtime spray
author: hans_l'hoest
---

Recently we developed a web client in Scala to start a Mendix application using only JSON commands similar as to how [m2ee-tools](https://github.com/mendix/m2ee-tools/) works.
In this post I will explain how to convert nested `Map`s in Scala to JSON using the [spray-json](https://github.com/spray/spray-json) library.

It is a basic example to show how easy it is the convert your own types to JSON using the default infrastructure provided by spray-json and
at the same time works around an issue in the library where nested `Map`s are not supported. The source code is also included 
as [a fully functional `sbt` project](https://github.com/lhohan/spray-json-pg) as well.

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

The `params` section is used to pass arguments for any potential action we support or may support in the future.
Meaning it can contain a hierarchical structure.

In Scala code we represent this structure not surprisingly with a `Map`:

```scala
Map("action" -> "action_name", "param2" -> 2, "param3" -> Map("param4" -> "value4"))
```

To marshall Scala objects to JSON we are using an easy to use, light-weight library called [spray-json](https://github.com/spray/spray-json).
There are efforts on its way to unify some of the Scala Json libraries currently available but until that time spray-json will do nicely. 
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

(1): the expected result all in a one liner for convenience so we do not have to worry about new lines etc. for verifying the result.

(2): `toJson` converts to an JSON AST (see [spray-json](https://github.com/spray/spray-json) doc which explains this very clearly) and calling toString gives us a plain string with the JSON output.

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
We return a `JsObject` initialized with the result of mapping each element (2) meanwhile converting each `Map` value as needed to its appropriate `JsValue` object.
As you can see, we explicitly convert `String` and `Int` (3).  
Type `Map[String, Any]` (4) is handled recursively: this the main construct which makes our conversion work. 
All other types we simply convert to a string (5); a further improvement could be to convert *any* type (say by using `toJson`), 
but this suffices to make our test pass and JSON requests work in practice.

The read part we do not need, so we leave it unimplemented (6).     

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

Code project with failing test is [here] (https://github.com/lhohan/spray-json-pg/tree/e2cda87713d6f835d756acafe9fc8f247cb23b4b).

Code project with the fix is [here] (https://github.com/lhohan/spray-json-pg/tree/84dd18227e8a6c488a502e8dbab0d0acfc0cddfd).

This concludes the main part of this post on our Scala classes into JSON. 
(If you want to get rid of the compiler warning, keep on reading.)

In case of questions, suggestions or additions please to not hesitate to leave a comment or contact me.

Addendum: removing the warning
---------

Off topic but you probably noticed the warning: 

```scala
[warn] .../src/main/scala/M2eeJsonProtocol.scala:10: non-variable type argument 
String in type pattern Map[String,Any] is unchecked since it is eliminated by 
erasure
[warn]           case v: Map[String, Any] => (k, write(v))                    // 4
[warn]                   ^
[warn] one warning found
```

As Scala is running on the jvm types are erased at runtime and the case pattern at (4) will not only match the `Map[String, Any]` type but *any* `Map` type.
Since we know we are only matching for `Map`s with keys of type `String` we are not looking to deal with other typed `Map`s, if this would 
happen it may crash with e.g. a ClassCastException but that's OK too, we'd rather crash hard in such cases. If we would want
to cover our bases we might even write an extra test to capture this behaviour but that is not our goal now. We just want to
get rid of the compiler warning.

There are various ways of removing this warning and it can get more complicated pretty fast: 

  - Matching on `Map[_,_]`, with or without checking all keys for `String` type, and make an explicit cast using `asInstanceOf` in the recursive call.
  - using manifest (but deprecated in Scala 2.10) 
  - using scalaz
  - using `TypeTag`s
  - using `scala.reflect.runtime` but this is [strongly discouraged](http://www.scala-lang.org/news/2.11.1) (which I find somewhat confusing)

Most suggestions can be investigated more through following [stackoverflow question](http://stackoverflow.com/questions/1094173/how-do-i-get-around-type-erasure-on-scala-or-why-cant-i-get-the-type-paramete).

Ideally I would like [this Scala improvement](https://issues.scala-lang.org/browse/SI-6517) but a rather easy fix is this:

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

But doesn't this all seem to be nothing but a rephrasing, some syntactic rearrangement, of the same thing?
It is and the above *only works in Scala 2.10* but when upgrading to Scala 2.11 the compiler has become smarter and figured it out:

```scala
[warn] .../src/main/scala/M2eeJsonProtocol.scala:12: non-variable type argument 
String in type pattern  scala.collection.immutable.Map[String,Any] (the 
underlying of M2eeJsonProtocol.StringToAny) is unchecked since it is eliminated 
by erasure
[warn]         case v: StringToAny => write(v)  // 4
[warn]                 ^
[warn] one warning found
```

Our [quick, relatively straight-forward, however not-the-most-elegant fix](https://github.com/lhohan/spray-json-pg/tree/bd518fd7e0217ac3b5473aa4b016083826b744ff) then would be to replace (4) with:

```scala
case v: Map[_, _] => write(v.asInstanceOf[Map[String, Any]])  // 4
```

This removes the warning in both Scala 2.10 and 2.11. If one day we revisit this item and a more elegant solution
comes up we'll keep you posted.
