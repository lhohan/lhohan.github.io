---
layout: post
title:  "non-variable type argument String in type pattern Map[String,Any] is unchecked since it is eliminated by erasure"
category: scala
categories: scala
author: hans_l'hoest
---

So how do we fix? : 

```scala
[warn] /Users/hans/dev/github/spray-json-pg/src/main/scala/M2eeJsonProtocol.scala:11: non-variable type argument String in type pattern Map[String,Any] is unchecked since it is eliminated by erasure
[warn]           case v: Map[String, Any] => (k, write(v))                    // 4
[warn]                   ^
[warn] one warning found
```

First lets understand what's going on:

The warning is pretty clear: types will be erased at runtime, as a result this case will also match other `Map` types, which obviously in general is not what we would want.
In our case there is not much of an issue in our JSON objects the keys in our maps will be `String`s but as we do not like warnings: here is how we fix it:


