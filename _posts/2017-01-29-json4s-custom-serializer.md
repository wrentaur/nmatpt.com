---
layout: post
title: Json4s custom serializers - the right way
---

Most examples of Json4s custom serializers use the following approach:

```scala
case class Animal(name: String, nrLegs: Int)

class AnimalSerializer extends CustomSerializer[Animal](format => ( {
  case JObject(
  JField("name", JString(name)) ::
    JField("nrLegs", JInt(nrLegs)) ::
    Nil
  ) => Animal(name, nrLegs.toInt)
}, {
  case animal: Animal =>
    ("name" -> animal.name) ~
      ("nrLegs" -> animal.nrLegs)
}
))
```

While this may sometimes work, **it is not the correct way to deserialize JSON**. The above code assumes the JSON is ordered, which is an incorrect assumption. A JSON object is, [per definition](http://json.org/), an unordered set of key/value pairs. Here is an example that breaks the deserializer:

``` diff

scala> implicit val fmt = org.json4s.DefaultFormats + new AnimalSerializer()

scala> val cat = """{"name":"cat", "nrLegs": 42}"""

scala> val catLegsFirst = """{"nrLegs": 42, "name": "cat"}"""

scala> read[Animal](cat)
res2: Animal = Animal(cat,42)

scala> read[Animal](catLegsFirst)
org.json4s.package$MappingException: Can't convert JObject(List((nrLegs,JInt(42)), (name,JString(cat)))) to class Animal
  at org.json4s.CustomSerializer$$anonfun$deserialize$1.applyOrElse(Formats.scala:373)
  at org.json4s.CustomSerializer$$anonfun$deserialize$1.applyOrElse(Formats.scala:370)
  at scala.runtime.AbstractPartialFunction.apply(AbstractPartialFunction.scala:36)
  at scala.PartialFunction$class.applyOrElse(PartialFunction.scala:123)
  at scala.collection.AbstractMap.applyOrElse(Map.scala:59)
  at scala.PartialFunction$OrElse.apply(PartialFunction.scala:167)
  at org.json4s.Extraction$.org$json4s$Extraction$$customOrElse(Extraction.scala:523)
  at org.json4s.Extraction$ClassInstanceBuilder.result(Extraction.scala:512)
  at org.json4s.Extraction$.extract(Extraction.scala:351)
  at org.json4s.Extraction$.extract(Extraction.scala:42)
  at org.json4s.ExtractableJsonAstNode.extract(ExtractableJsonAstNode.scala:21)
  at org.json4s.jackson.Serialization$.read(Serialization.scala:50)
  ... 43 elided

```

If your application receives JSON from an external system, you can not predict the order of the received data and you should not rely on it. Here is an implementation that works regardless of the order. Note that the serializer code is the same as above, which is correct, I just changed the deserialization logic:

``` scala
import org.json4s.CustomSerializer
import org.json4s.JsonAST._
import org.json4s.JsonDSL._

case class Animal(name: String, nrLegs: Int)

class AnimalSerializerUnordered extends CustomSerializer[Animal](format => ( {
  case jsonObj: JObject =>
    val name = (jsonObj \ "name").extract[String]
    val nrLegs = (jsonObj \ "nrLegs").extract[Int]

    Animal(name, nrLegs)
}, {
  case animal: Animal =>
    ("name" -> animal.name) ~
      ("nrLegs" -> animal.nrLegs)
}
))

```
Since each value is extracted independently, the order of the original object does not matter. You can see more details about Json4s value extraction [here](http://json4s.org/#extracting-values).
