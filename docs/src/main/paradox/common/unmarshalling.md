# 解组

"解组"是将某种较低级别的表示形式(通常是"wire格式")转换为较高级别(对象)结构的过程。它的其他流行名称是"反序列化"或"Unpickling"。

在Akka HTTP中，"解组"是指将较低级别的源对象，例如，一个`MessageEntity`(其构成HTTP请求或响应的"entity body")或一个完整的 @apidoc[HttpRequest] 或 @apidoc[HttpResponse]，转换为`T`类型的实例。

## 基本设计

将类型`A`实例解组为类型`B`实例是由一个 @apidoc[Unmarshaller[A, B]] 执行的。

@@@ div { .group-scala }
Akka HTTP还为您可能会使用的大多数解组器类型预定义了许多有用的别名：
@@snip [package.scala]($akka-http$/akka-http/src/main/scala/akka/http/scaladsl/unmarshalling/package.scala) { #unmarshaller-aliases }

@@@

在它的核心， @apidoc[Unmarshaller[A, B]] 与函数 @scala[function `A => Future[B]`]@java[`Function<A, CompletionStage<B>>`] 非常相似，因此比其编组对应的 @ref[marshalling](marshalling.md) 对象要简单得多。解组过程不必支持内容协商，这节省了编组端所需的两个额外的间接层。

## 使用解组器

有关如何在服务器端使用解组器的示例，请参见 @ref[动态路由示例](../routing-dsl/index.md#动态路由示例)。对于客户端，请参阅@ref[处理响应](../client-side/request-and-response.md#processing-responses)。

## 预定义的解组器

Akka HTTP已经为最常见的类型预定义了许多解组器。具体来说是：

 * @scala[@scaladoc[PredefinedFromStringUnmarshallers](akka.http.scaladsl.unmarshalling.PredefinedFromStringUnmarshallers)]
   @java[@javadoc[StringUnmarshallers](akka.http.javadsl.unmarshalling.StringUnmarshallers)]
    * `Byte`
    * `Short`
    * @scala[`Int`]@java[`Integer`]
    * `Long`
    * `Float`
    * `Double`
    * `Boolean`
 * @scala[@scaladoc[PredefinedFromEntityUnmarshallers](akka.http.scaladsl.unmarshalling.PredefinedFromEntityUnmarshallers)]
   @java[@apidoc[Unmarshaller]]
    * @scala[`Array[Byte]`]@java[`byte[]`]
    * @apidoc[akka.util.ByteString]
    * @scala[`Array[Char]`]@java[`char[]`]
    * `String`
    * @scala[`akka.http.scaladsl.model.FormData`]@java[`akka.http.javadsl.model.FormData`]

@@@ div { .group-scala }
 * @scaladoc[GenericUnmarshallers](akka.http.scaladsl.unmarshalling.GenericUnmarshallers)
    * @apidoc[Unmarshaller[T, T]] (identity unmarshaller)
    * @apidoc[Unmarshaller[Option[A], B]], if an @apidoc[Unmarshaller[A, B]] is available
    * @apidoc[Unmarshaller[A, Option[B]]], if an @apidoc[Unmarshaller[A, B]] is available
@@@

其他解组器可在单独的模块中用于特定的内容类型，例如 @ref[JSON](json-support.md)@scala[和 @ref[XML](xml-support.md)]。

@@@ div { .group-scala }

## 隐式解决

Akka HTTP的解组基础结构依赖于基于类型类(type-class)的方法，这意味着从特定类型`A`到特定类型`B`的 @apidoc[Unmarshaller] 实例必须隐式可用。

Akka HTTP中大多数预定义的解组器的隐式都是通过 @apidoc[Unmarshaller] 特质的伴生对象提供的。这意味着它们始终可用，无需显式导入。另外，您可以通过将自己的自定义版本引入本地作用域来简单地“覆盖”它们。

@@@

## 自定义解组器

Akka HTTP为您提供了一些方便的工具，用于为您自己的类型构造解组器。通常，您不必直接"手动"实现 @apidoc[Unmarshaller] @scala[特质]@java[类]。相反，它应该可以使用在 @scala[the @apidoc[Unmarshaller]伴生对象]@java[@apidoc[Unmarshaller]]上定义的方便的构造助手：

Scala
:  @@snip [Unmarshaller.scala]($akka-http$/akka-http/src/main/scala/akka/http/scaladsl/unmarshalling/Unmarshaller.scala) { #unmarshaller-creation }

Java
:  @@snip [Unmarshallers.scala]($akka-http$/akka-http/src/main/java/akka/http/javadsl/unmarshalling/Unmarshallers.java) { #unmarshaller-creation }

@@@ note
为了避免不必要的内存压力，解组器应确保完全消耗传入的实体数据流，或确保在出错时将其正确取消。否则，可能会将流的其余部分保留在内存中的时间超过必要的时间。
@@@

## 派生解组器

有时，您可以通过将现有的解组器重用于一个自定义解组器来节省一些工作。这个想法是用某种逻辑“包装”现有的解组器，以将其“重新定位”到您的类型。

通常，您要做的是转换一些现有解组器的输出并将其转换为您的类型。对于此类解组器转换，Akka HTTP定义了以下方法：

@@@ div { .group-scala }
 * `baseUnmarshaller.transform`
 * `baseUnmarshaller.map`
 * `baseUnmarshaller.mapWithInput`
 * `baseUnmarshaller.flatMap`
 * `baseUnmarshaller.flatMapWithInput`
 * `baseUnmarshaller.recover`
 * `baseUnmarshaller.withDefaultValue`
 * `baseUnmarshaller.mapWithCharset` (only available for FromEntityUnmarshallers)
 * `baseUnmarshaller.forContentTypes` (only available for FromEntityUnmarshallers)
@@@

@@@ div { .group-java }
 * `baseMarshaller.thenApply`
 * `baseMarshaller.flatMap`
 * `Unmarshaller.forMediaType` (to derive from a @apidoc[HttpEntity] unmarshaller)
 * `Unmarshaller.forMediaTypes` (to derive from a @apidoc[HttpEntity] unmarshaller)
@@@

方法签名应该使它们的语义相对清晰。

## 使用解组器

在Akka的许多地方，HTTP解组器都是隐式使用的，例如，当您想使用 @ref[路由DSL](../routing-dsl/index.md) 访问请求的 @ref[实体](../routing-dsl/directives/marshalling-directives/entity.md) 时。

但是，您也可以根据需要直接使用解组基础结构，这在测试中很有用。它的最好的入口点是 @scala[`akka.http.scaladsl.unmarshalling.Unmarshal`对象]@java[`akka.http.javadsl.unmarshalling.StringUnmarshallers` class]，你可以这样使用：

Scala
:  @@snip [UnmarshalSpec.scala]($test$/scala/docs/http/scaladsl/UnmarshalSpec.scala) { #use-unmarshal }

Java
:  @@snip [UnmarshalTest.scala]($test$/java/docs/http/javadsl/UnmarshalTest.java) { #imports #use-unmarshal }
