# 编组

@java[TODO @github[overhaul for Java](#1367)]

编组是将高级别(对象)结构转换为某种低级别表示形式的过程，通常是"wire格式"。编组的其他流行名称是"序列化"或"pickle"。

在Akka HTTP中，编组意味着将`T`类型的对象转换为较低级别的目标类型，例如，一个`MessageEntity`(形成HTTP请求或响应的"消息体")或一个完整的 @apidoc[HttpRequest] 或 @apidoc[HttpResponse]。

例如，在服务器端，编组用于将应用程序域(application-domain)对象转换为响应实体。请求可以包含一个 @apidoc[Accept] 标头，它列出了客户端可接受的内容类型，例如`application/json`和`application/xml`。编组器包含根据 @apidoc[Accept] 和`AcceptCharset`标头协商结果内容类型的逻辑。

## 基本设计

将类型`A`的实例编组为类型`B`的实例是由 @apidoc[Marshaller[A, B]] 执行的。

与您最初期望的相反，@apidoc[Marshaller[A, B]]不是一个简单的`A => B`函数，而本质上是一个 @scala[`A => Future[List[Marshalling[B]]]`]@java[`A => CompletionStage<List<Marshalling<B>>>`] 函数。让我们逐一剖析这个看起来相当复杂的签名，以了解为何以这种方式设计编组器。给类型`A`的实例一个 @apidoc[Marshaller[A, B]] 产生：

1. 一个 @scala[`Future`]@java[`CompletionStage`]：这可能是相当清楚的。编组不需要同步产生一个结果，因此，它们返回一个future，这允许编组过程中的异步性。

2. of `List`: `A`编组器可以提供多个目标表示，而不是一个单一的。最终由内容协商决定将渲染哪一个。例如，@apidoc[Marshaller[OrderConfirmation, MessageEntity]] 可能提供JSON以及XML表示形式。客户端可以通过附加的 @apidoc[Accept] 请求标头来决定首选哪一个。如果客户机没有表示首选项，则选择第一个表示。

3. of @scala[`Marshalling[B]`]@java[`Marshalling<B>`]：不是直接返回`B`的实例，编组器首先产生一个 @scala[`Marshalling[B]`]@java[`Marshalling<B>`]。这允许查询 @apidoc[MediaType] 和潜在的 @apidoc[HttpCharset]，其将在编组器将在触发实际编组之前生成。除了启用内容协商之外，此设计还允许将编组目标实例的实际构造延迟到真正需要它的最后时刻。
 
@@@ div { .group-scala }

这是`Marshalling`如何定义的：

@@snip [Marshaller.scala]($akka-http$/akka-http/src/main/scala/akka/http/scaladsl/marshalling/Marshaller.scala) { #marshalling }

Akka HTTP还为您可能最常使用的编组器类型定义了许多有用的别名:

@@snip [package.scala]($akka-http$/akka-http/src/main/scala/akka/http/scaladsl/marshalling/package.scala) { #marshaller-aliases }

@@@

## 预定义的编组器

Akka HTTP已经为最常见的类型预定义了许多编组器。具体来说是：

@@@ div { .group-scala }

 * @scaladoc[PredefinedToEntityMarshallers](akka.http.scaladsl.marshalling.PredefinedToEntityMarshallers)
    * `Array[Byte]`
    * @apidoc[akka.util.ByteString]
    * `Array[Char]`
    * `String`
    * `akka.http.scaladsl.model.FormData`
    * `akka.http.scaladsl.model.MessageEntity`
    * `T <: akka.http.scaladsl.model.Multipart`
 * @scaladoc[PredefinedToResponseMarshallers](akka.http.scaladsl.marshalling.PredefinedToResponseMarshallers)
    * `T`, if a `ToEntityMarshaller[T]` is available
    * @apidoc[HttpResponse]
    * @apidoc[StatusCode]
    * `(StatusCode, T)`, if a `ToEntityMarshaller[T]` is available
    * `(Int, T)`, if a `ToEntityMarshaller[T]` is available
    * `(StatusCode, immutable.Seq[HttpHeader], T)`, if a `ToEntityMarshaller[T]` is available
    * `(Int, immutable.Seq[HttpHeader], T)`, if a `ToEntityMarshaller[T]` is available
 * @scaladoc[PredefinedToRequestMarshallers](akka.http.scaladsl.marshalling.PredefinedToRequestMarshallers)
    * @apidoc[HttpRequest]
    * @apidoc[Uri]
    * `(HttpMethod, Uri, T)`, if a `ToEntityMarshaller[T]` is available
    * `(HttpMethod, Uri, immutable.Seq[HttpHeader], T)`, if a `ToEntityMarshaller[T]` is available
 * @scaladoc[GenericMarshallers](akka.http.scaladsl.marshalling.GenericMarshallers)
    * @apidoc[Marshaller[Throwable, T]]
    * @apidoc[Marshaller[Option[A], B]], if a @apidoc[Marshaller[A, B]] and an `EmptyValue[B]` is available
    * @apidoc[Marshaller[Either[A1, A2], B]], if a @apidoc[Marshaller[A1, B]] and a @apidoc[Marshaller[A2, B]] is available
    * @apidoc[Marshaller[Future[A], B]], if a @apidoc[Marshaller[A, B]] is available
    * @apidoc[Marshaller[Try[A], B]], if a @apidoc[Marshaller[A, B]] is available

@@@

@@@ div { .group-java }

 * Predefined @apidoc[RequestEntity] marshallers:
    * `byte[]`
    * @apidoc[akka.util.ByteString]
    * `char[]`
    * `String`
    * @apidoc[FormData]
    * `Optional<T>` using an existing @apidoc[RequestEntity] marshaller for `T`. An empty optional will yield an empty entity.
 * Predefined @apidoc[HttpResponse] marshallers:
    * `T` using an existing @apidoc[RequestEntity] marshaller for `T`
    * `T` and @apidoc[StatusCode] using an existing @apidoc[RequestEntity] marshaller for `T`
    * `T`, @apidoc[StatusCode] and `Iterable[HttpHeader]` using an existing @apidoc[RequestEntity] marshaller for `T`

All marshallers can be found in @apidoc[Marshaller].

@@@

@@@ div { .group-scala }

## 隐式解决

Akka HTTP的编组基础结构依赖于基于类型类(type-class)的方法，这意味着 @apidoc[Marshaller] 实例从某种类型`A`到某种类型`B`必须是隐式可用。

Akka HTTP中大多数预定义的编组器的隐式是通过 @apidoc[Marshaller] 特质的伴生对象提供的。这意味着它们始终可用，无需显式导入。另外，您可以通过将自己的自定义版本引入本地作用域来简单地“覆盖”它们。

@@@

## 自定义编组器

Akka HTTP为您提供了一些方便的工具，用于为您自己的类型构造编组器。在执行此操作之前，您需要考虑要你想要创建哪种编组器。如果您的所有编组需要生成一个`MessageEntity`，那么您可能应该提供一个  @scala[`ToEntityMarshaller[T]`]@java[@apidoc[Marshaller[T, MessageEntity]]]。这样做的优点是，它会同时在客户端以及服务器端工作，因为一个 @scala[`ToReponseMarshaller[T]`]@java[@apidoc[Marshaller[T, HttpResponse]]] 和一个 @scala[`ToRequestMarshaller[T]`]@java[@apidoc[Marshaller[T, HttpRequest]]] 可以自动的创建，如果一个 @scala[`ToEntityMarshaller[T]`]@java[@apidoc[Marshaller[T, MessageEntity]]] 是可用的。

但是，如果您的编组还需要设置诸如响应状态码，请求方法，请求URI或任何标头之类的内容，则 @scala[`ToEntityMarshaller[T]`]@java[@apidoc[Marshaller[T, MessageEntity]]] 将不能工作。您需要直接提供一个 @scala[`ToResponseMarshaller[T]`]@java[@apidoc[Marshaller[T, HttpResponse]]] 或一个@scala[`ToRequestMarshaller[T]]`]@java[@apidoc[Marshaller[T, HttpRequest]]]。

要编写自己的编组器，您不必直接“手动”实现 @apidoc[Marshaller] @scala[特质]@java[类]。

@@@ div { .group-scala }

相反，应该可以使用一个定义在 @apidoc[Marshaller] 伴生对象中的便捷构造助手：

@@snip [Marshaller.scala]($akka-http$/akka-http/src/main/scala/akka/http/scaladsl/marshalling/Marshaller.scala) { #marshaller-creation }

@@@

## 派生编组器

有时，您可以通过将现有的编组重用为自定义编组来节省一些工作。这个想法是用某种逻辑"包装"现有的编组器，以将其"重定向"到您的类型。

在这方面，将编组程序包装起来可能意味着以下两件事之一或全部：

 * 在输入到达包装的编组器之前进行转换
 * 转换包装的编组器的输出

对于后者(转换输出)，您可以使用`baseMarshaller.map`，其工作恰好与它为函数做的一样。对于前者(转换输入)，您有四个选择：

 * `baseMarshaller.compose`
 * `baseMarshaller.composeWithEC`
 * `baseMarshaller.wrap`
 * `baseMarshaller.wrapWithEC`

`compose`的工作恰好与它为函数做的一样。`wrap`是一个组合，也允许您更改`ContentType`，这正是编组器要编组的。这些`...WithEC`变体允许您在在内部接收一个`ExecutionContext`，如果需要一个的话，而不必依赖使用现场隐式可用的一种。

## 使用编组器

在Akka HTTP的许多地方，都隐式使用了编组，例如，当您定义如何使用 @ref[路由DSL](../routing-dsl/index.md) @ref[完成](../routing-dsl/directives/route-directives/complete.md) 请求时。

@@@ div { .group-scala }

但是，您也可以根据需要直接使用编组基础结构，这在测试中很有用。最好的入口点是 @scaladoc[Marshal](akka.http.scaladsl.marshalling.Marshal) 对象，您可以像这样使用它：

@@snip [MarshalSpec.scala]($test$/scala/docs/http/scaladsl/MarshalSpec.scala) { #use-marshal }

@@@

@@@ div { .group-java }

However, many directives dealing with @ref[marshalling](../routing-dsl/directives/marshalling-directives/index.md) also  require that you pass a marshaller explicitly. The following example shows how to marshal Java bean classes to JSON using the @ref:[Jackson JSON support](json-support.md#jackson-support):

@@snip [PetStoreExample.java]($akka-http$/akka-http-tests/src/main/java/akka/http/javadsl/server/examples/petstore/PetStoreExample.java) { #imports #marshall }

@@@
