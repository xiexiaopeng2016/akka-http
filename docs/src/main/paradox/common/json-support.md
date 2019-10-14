# JSON支持

Akka HTTP的 @ref[编组](marshalling.md) 和 @ref[解组](unmarshalling.md) 基础设施使得它比较容易无缝转换应用程序域对象从和到JSON。@scala[`akka-http-spray-json`]@java[`akka-http-jackson`] 模块开箱即用地提供了与 @scala[[spray-json]]@java[[Jackson]] 的集成。与其他JSON库的集成是由社区支持的。请参阅[Akka HTTP的当前社区扩展列表](https://akka.io/community/#extensions-to-akka-http)。

@@@ div { .group-java }

## Jackson Support

To make use of the support module for (un)marshalling from and to JSON with [Jackson], add a library dependency onto:

@@dependency [sbt,Gradle,Maven] {
  group="com.typesafe.akka"
  artifact="akka-http-jackson_$scala.binary.version$"
  version="$project.version$"
}

Use `akka.http.javadsl.marshallers.jackson.Jackson.unmarshaller(T.class)` to create an @apidoc[Unmarshaller[HttpEntity,T]] which expects the request
body (HttpEntity) to be of type `application/json` and converts it to `T` using Jackson.

@@snip [PetStoreExample.java]($akka-http$/akka-http-tests/src/main/java/akka/http/javadsl/server/examples/petstore/PetStoreExample.java) { #imports #unmarshall }

Use `akka.http.javadsl.marshallers.jackson.Jackson.marshaller(T.class)` to create a @apidoc[Marshaller[T,RequestEntity]] which can be used with
`RequestContext.complete` or `RouteDirectives.complete` to convert a POJO to an HttpResponse.

@@snip [PetStoreExample.java]($akka-http$/akka-http-tests/src/main/java/akka/http/javadsl/server/examples/petstore/PetStoreExample.java) { #imports #marshall }

Refer to @github[this file](/akka-http-tests/src/main/java/akka/http/javadsl/server/examples/petstore/PetStoreExample.java) in the sources for the complete example.

@@@


@@@ div { .group-scala }

## spray-json支持

@scaladoc[SprayJsonSupport](akka.http.scaladsl.marshallers.sprayjson.SprayJsonSupport)特质提供了一个`FromEntityUnmarshaller[T]`和`ToEntityMarshaller[T]`为每一个类型`T`，一个隐式`spray.json.RootJsonReader`和/或`spray.json.RootJsonWriter`(分别)是可用的。

要启用自动支持使用[spray-json]从和到JSON编(解)组，请将库依赖项添加到：

@@dependency [sbt,Gradle,Maven] {
  group="com.typesafe.akka"
  artifact="akka-http-spray-json_$scala.binary.version$"
  version="$project.version$"
}

接下来，为您的类型提供一个`RootJsonFormat[T]`并将其纳入到作用域。请查看[spray-json]文档，以获取有关如何执行此操作的更多信息。

最后，直接从`SprayJsonSupport`中导入`FromEntityUnmarshaller[T]`和`ToEntityMarshaller[T]`隐式，如下面的示例所示，或将`akka.http.scaladsl.marshallers.sprayjson.SprayJsonSupport`特质混入到您的JSON支持模块中。

一旦你完成了JSON和类型`T`之间的编排(解组)，类型`T`应该可以正常且透明地进行工作。

@@snip [SprayJsonExampleSpec.scala]($test$/scala/docs/http/scaladsl/SprayJsonExampleSpec.scala) { #minimal-spray-json-example }

@@@ 

<a id="json-streaming-client-side"></a>
## 消费JSON流样式API

实现流API的一种流行方法是[JSON流](https://en.wikipedia.org/wiki/JSON_Streaming)(有关构建此类服务器端API的文档，请参阅@ref[源流](../routing-dsl/source-streaming-support.md))。

根据API返回流式JSON（换行符分隔，对象的原始序列或“无限数组”）的方式，您可能必须应用不同的框架机制，但总体思路仍然相同：使用无限实体流并应用框架，以便可以使用常规的编组基础结构轻松地对单个对象进行反序列化：

Depending on the way the API returns the streamed JSON (newline delimited, raw sequence of objects, or "infinite array") you may have to apply a different framing mechanism, but the general idea remains the same: consuming the infinite entity stream and applying a framing to it, such that the single objects can be easily deserialized using the usual marshalling infrastructure:

Scala
:   @@snip [EntityStreamingSpec.scala]($akka-http$/akka-http-tests/src/test/scala/akka/http/scaladsl/server/EntityStreamingSpec.scala) { #json-streaming-client-example }
 
Java
:   @@snip [HttpClientExampleDocTest.java]($test$/java/docs/http/javadsl/server/JsonStreamingExamplesTest.java) { #json-streaming-client-example-raw }

@@@ div { .group-scala }

在上面的示例中，编组由隐式提供的`JsonEntityStreamingSupport`进行处理，它也用在构建服务器端流式API时。您还可以更明确地实现相同的功能，通过手动连接实体字节流通过一个帧，然后反序列化阶段:

Scala
:   @@snip [EntityStreamingSpec.scala]($akka-http$/akka-http-tests/src/test/scala/akka/http/scaladsl/server/EntityStreamingSpec.scala) { #json-streaming-client-example-raw }
 
@@@

@@@ div { .group-java }

In the above example the `JsonEntityStreamingSupport` class is used to obtain the proper framing, though you could also
pick the framing manually by using `akka.stream.javadsl.Framing` or `akka.stream.javadsl.JsonFraming`. 
Framing stages are used to "chunk up" the pieces of incoming bytes into appropriately sized pieces of valid JSON,
which then can be handled easily by a not-streaming JSON serializer such as jackson in the example. This technique is simpler to use
and often good enough rather than writing a fully streaming JSON parser (which also is possible). 

@@@ 


@@@ div { .group-scala }

## 漂亮的打印

默认情况下，Spray-json通过使用`CompactPrinter`进行隐式转换，将您的类型编组为紧凑的打印JSON ，如以下定义：

@@snip [SprayJsonSupport.scala]($akka-http$/akka-http-marshallers-scala/akka-http-spray-json/src/main/scala/akka/http/scaladsl/marshallers/sprayjson/SprayJsonSupport.scala) { #sprayJsonMarshallerConverter }

另外，也可以将您的类型编组为漂亮的打印JSON，并引入`PrettyPrinter`到作用域以执行隐式转换。

@@snip [SprayJsonPrettyMarshalSpec.scala]($test$/scala/docs/http/scaladsl/SprayJsonPrettyMarshalSpec.scala) { #example }

要了解有关spray-json如何工作的更多信息，请参阅其[文档][spray-json]。

@@@

[spray-json]: https://github.com/spray/spray-json
[jackson]: https://github.com/FasterXML/jackson
