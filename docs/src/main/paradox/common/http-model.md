# HTTP模型

Akka HTTP模型包含一个结构深刻的，完全不变的，基于样例类的模型，该模型包含所有主要的HTTP数据结构，例如HTTP请求，响应和公共报头。它位于 *akka-http-core* 模块中，并构成了大多数Akka HTTP API的基础。

## 概述

由于akka-http-core提供了主要的HTTP数据结构，因此您会在代码库的很多地方找到以下导入(也可能是您自己的代码)：

Scala
:   @@snip [ModelSpec.scala]($test$/scala/docs/http/scaladsl/ModelSpec.scala) { #import-model }

Java
:   @@snip [ModelDocTest.java]($test$/java/docs/http/javadsl/ModelDocTest.java) { #import-model }

这带来了作用域内所有最相关的类型，主要是：

 * @apidoc[HttpRequest] 和 @apidoc[HttpResponse], 主要的消息模型
 * `headers`, 该软件包包含所有预定义的HTTP报头模型和支持类型
 * 支持类型，例如 @apidoc[Uri], @apidoc[HttpMethods$], @apidoc[MediaTypes$], @apidoc[StatusCodes$], 等。

一种常见的模式是某个实体的模型由不可变的类型(类或特质)表示，而HTTP规范定义的实体的实际实例位于一个伴随对象中，该对象带有该类型的名称以及结尾的复数形式's'。

例如:

 * 定义的 @apidoc[HttpMethod]实例 @scala[位于]@java[are defined as static fields of] @apidoc[HttpMethods$] @scala[对象]@java[class].
 * 定义的 @apidoc[HttpCharset]实例 @scala[位于]@java[are defined as static fields of] @apidoc[HttpCharsets$] @scala[对象]@java[class].
 * 定义的 @apidoc[HttpEncoding]实例 @scala[位于]@java[are defined as static fields of] @apidoc[HttpEncodings$] @scala[对象]@java[class].
 * 定义的 @apidoc[HttpProtocol]实例 @scala[位于]@java[are defined as static fields of] @apidoc[HttpProtocols$] @scala[对象]@java[class].
 * 定义的 @apidoc[MediaType]实例 @scala[位于]@java[are defined as static fields of] @apidoc[MediaTypes$] @scala[对象]@java[class].
 * 定义的 @apidoc[StatusCode]实例 @scala[位于]@java[are defined as static fields of] @apidoc[StatusCodes$] @scala[对象]@java[class].

## HttpRequest

@apidoc[HttpRequest] 和 @apidoc[HttpResponse]是表示HTTP消息的基本 @scala[样例]@java[immutable] 类。

一个 @apidoc[HttpRequest] 由以下组成

 * 一个方法(GET, POST, 等)
 * 一个URI (查阅 @ref[URI model](uri-model.md) 了解更多信息)
 * 一连串的标头
 * 一个实体(主体数据)
 * 一个协议

以下是一些如何构造 @apidoc[HttpRequest] 的示例：

Scala
:   @@snip [ModelSpec.scala]($test$/scala/docs/http/scaladsl/ModelSpec.scala) { #construct-request }

Java
:   @@snip [ModelDocTest.java]($test$/java/docs/http/javadsl/ModelDocTest.java) { #construct-request }

@@@ div { .group-scala }
`HttpRequest.apply`的所有参数都设置了默认值，因此`headers`，例如，如果它是none，则无需指定。许多参数类型(例如 @apidoc[HttpEntity]和 @apidoc[Uri])为常见用例定义了隐式转换，以简化请求和响应实例的创建。
@@@
@@@ div { .group-java }
In its basic form `HttpRequest.create` creates an empty default GET request without headers which can then be
transformed using one of the `withX` methods, `addHeader`, or `addHeaders`. Each of those will create a
new immutable instance, so instances can be shared freely. There exist some overloads for `HttpRequest.create` that
simplify creating requests for common cases. Also, to aid readability, there are predefined alternatives for `create`
named after HTTP methods to create a request with a given method and URI directly.
@@@

<a id="synthetic-headers"></a>
### 合成报头

在某些情况下，可能有必要偏离完全符合RFC的行为。例如，Amazon S3将URL路径部分中的`+`字符视为空格，即使RFC规定此行为应仅限于URI的查询部分。

为了解决这些类型的极端情况，Akka HTTP提供了通过合成标头向请求提供额外的非标准信息的能力。这些标头不传递给客户端，而是由请求引擎消费，并用于覆盖默认行为。

例如，为了提供原始请求uri，绕过默认的URL规范化，您可以执行以下操作：


Scala
:   @@snip [ModelSpec.scala]($test$/scala/docs/http/scaladsl/ModelSpec.scala) { #synthetic-header-s3 }

Java
:   @@snip [ModelDocTest.java]($test$/java/docs/http/javadsl/ModelDocTest.java) { #synthetic-header-s3 }

## HttpResponse

一个 @apidoc[HttpResponse] 对象包括

 * 一个状态码
 * 一个 @scala[`Seq`]@java[list] 报头
 * 一个实体(主体数据)
 * 一个协议

以下是一些如何构造 @apidoc[HttpResponse] 的示例：

Scala
:   @@snip [ModelSpec.scala]($test$/scala/docs/http/scaladsl/ModelSpec.scala) { #construct-response }

Java
:   @@snip [ModelDocTest.java]($test$/java/docs/http/javadsl/ModelDocTest.java) { #construct-response }

除了从固定`String`或 @apidoc[akka.util.ByteString] 创建实体的简单 @scala[@apidoc[HttpEntity]构造函数]@java[`HttpEntities.create` methods] 之外，Akka HTTP模型还定义了许多 @apidoc[HttpEntity] 子类，这些子类允许将主体数据指定为字节流。@java[All of these types can be created using the method on `HttpEntites`.]

<a id="httpentity"></a>
## HttpEntity

一个 @apidoc[HttpEntity] 携带消息的数据字节，连同Content-Type，如果已知的话，它的Content-Length。在Akka HTTP中，有五种不同的实体，它们可以模拟接收或发送消息内容的各种方式：

@scala[HttpEntity.Strict]@java[HttpEntityStrict]
: 最简单的实体，当所有实体在内存中都可用时使用。它包装了一个普通的 @apidoc[akka.util.ByteString]，并表示一个标准的，带有已知`Content-Length`的未分块(unchunked)实体。

@scala[HttpEntity.Default]@java[HttpEntityDefault]
: 常规的，未分块(unchunked) HTTP/1.1消息实体。它具有已知的长度，并将其数据展示为一个只能实现一次的 @apidoc[Source[ByteString, \_]] 。如果提供的源产生的字节数与指定的字节数不完全相同，则会出现错误。 @scala[`Strict`]@java[`HttpEntityStrict`] 和 @scala[`Default`]@java[`HttpEntityDefault`] 的差别是一个API-only one。在线上(wire)，两种实体看起来都相同。

@scala[HttpEntity.Chunked]@java[HttpEntityChunked]
: 针对HTTP/1.1 [分块内容](https://tools.ietf.org/html/rfc7230#section-4.1) (即带`Transfer-Encoding: chunked`发送) 的模型。内容长度未知，且单独的块被呈现为一个 @scala[`Source[HttpEntity.ChunkStreamPart]`]@java[@apidoc[Source[ChunkStreamPart, ?]]]。`ChunkStreamPart`是一个非空的 @scala[`Chunk`]@java[chunk] 或 @scala[一个`LastChunk`]@java[the empty last chunk] 包含可选的trailer报头。流由零个或多个 @scala[`Chunked`]@java[non-empty chunks] 部分组成，可由一个可选的 @scala[`LastChunk`部分]@java[last chunk] 终止。

@scala[HttpEntity.CloseDelimited]@java[HttpEntityCloseDelimited]
: 长度未知的未分块的实体，其通过关闭连接(`Connection: close`)隐式地分隔。内容数据表示为一个 @apidoc[Source[ByteString, \_]]。由于必须在发送这种类型的实体后关闭连接，所以只能在服务器端使用它来发送响应。此外，`CloseDelimited`实体的主要目的是与HTTP/1.0对等点(peers)兼容，后者不支持分块传输编码。如果您正在构建一个新的应用程序，并且没有受到遗留需求的限制，那么您不应该依赖于`CloseDelimited`实体，因为隐式的terminate-by-connection-close不是一种发送响应结束信号的可靠方式，尤其是在存在代理的情况下。此外，这种类型的实体可以防止连接重用，从而严重降低性能。使用 @scala[`HttpEntity.Chunked`]@java[`HttpEntityChunked`] 代替!

@scala[HttpEntity.IndefiniteLength]@java[HttpEntityIndefiniteLength]
: 长度未指定的流实体，在`Multipart.BodyPart`内使用。

实体类型 @scala[`Strict`]@java[`HttpEntityStrict`], @scala[`Default`]@java[`HttpEntityDefault`], 和 @scala[`Chunked`]@java[`HttpEntityChunked`] 是 @scala[`HttpEntity.Regular`]@java[@apidoc[RequestEntity]] 的子类型，这允许将它们用于请求和响应。与此相反， @scala[`HttpEntity.CloseDelimited`]@java[`HttpEntityCloseDelimited`] 只能用于响应。

流媒体实体类型(即除了@scala[`Strict`]@java[`HttpEntityStrict`])不能共享或序列化。使用`HttpEntity.toStrict` 或 `HttpMessage.toStrict`创建实体或消息的严格的、可共享的副本, 其返回一个对象的 @scala[`Future`]@java[`CompletionStage`], 具有收集到 @apidoc[akka.util.ByteString] 的主体数据。

@scala[@apidoc[HttpEntity]伴生对象]@java[class `HttpEntities`]包含 @scala[几个辅助构造函数]@java[static methods]从常见类型轻松地创建实体。

如果希望为每个子类型提供特殊处理，你可以在 @apidoc[HttpEntity] @java[to find out of which subclass an entity is] 的 @scala[子类型]@java[`isX` methods]之上做模式匹配。但是，在许多情况下， @apidoc[HttpEntity]的接收者并不关心实体是哪种子类型(以及如何在HTTP层上精确传输数据)。因此，提供了一种通用方法 @scala[`HttpEntity.dataBytes`]@java[`HttpEntity.getDataBytes()`]，该方法返回 @apidoc[Source[ByteString, \_]]，它允许访问实体的数据，而不管其具体的子类型。

@@@ note { title='什么时候使用那些子类型？' }

 * Use @scala[`Strict`]@java[`HttpEntityStrict`] if the amount of data is "small" and already available in memory (e.g. as a `String` or @apidoc[akka.util.ByteString])
 * Use @scala[`Default`]@java[`HttpEntityDefault`] if the data is generated by a streaming data source and the size of the data is known
 * Use @scala[`Chunked`]@java[`HttpEntityChunked`] for an entity of unknown length
 * Use @scala[`CloseDelimited`]@java[`HttpEntityCloseDelimited`] for a response as a legacy alternative to @scala[`Chunked`]@java[`HttpEntityChunked`] if the client
doesn't support chunked transfer encoding. Otherwise use @scala[`Chunked`]@java[`HttpEntityChunked`]!
 * In a `Multipart.BodyPart` use @scala[`IndefiniteLength`]@java[`HttpEntityIndefiniteLength`] for content of unknown length.

@@@

@@@ warning { title="Caution" }

When you receive a non-strict message from a connection then additional data are only read from the network when you
request them by consuming the entity data stream. This means that, if you *don't* consume the entity stream then the
connection will effectively be stalled. In particular no subsequent message (request or response) will be read from
the connection as the entity of the current message "blocks" the stream.
Therefore you must make sure that you always consume the entity data, even in the case that you are not actually
interested in it!

@@@

### Limiting message entity length

All message entities that Akka HTTP reads from the network automatically get a length verification check attached to
them. This check makes sure that the total entity size is less than or equal to the configured
`max-content-length` <a id="^1" href="#1">[1]</a>, which is an important defense against certain Denial-of-Service attacks.
However, a single global limit for all requests (or responses) is often too inflexible for applications that need to
allow large limits for *some* requests (or responses) but want to clamp down on all messages not belonging into that
group.

In order to give you maximum flexibility in defining entity size limits according to your needs the @apidoc[HttpEntity]
features a `withSizeLimit` method, which lets you adjust the globally configured maximum size for this particular
entity, be it to increase or decrease any previously set value.
This means that your application will receive all requests (or responses) from the HTTP layer, even the ones whose
`Content-Length` exceeds the configured limit (because you might want to increase the limit yourself).
Only when the actual data stream @apidoc[Source] contained in the entity is materialized will the boundary checks be
actually applied. In case the length verification fails the respective stream will be terminated with an
`EntityStreamSizeException` either directly at materialization time (if the `Content-Length` is known) or whenever more
data bytes than allowed have been read.

When called on `Strict` entities the `withSizeLimit` method will return the entity itself if the length is within
the bound, otherwise a `Default` entity with a single element data stream. This allows for potential refinement of the
entity size limit at a later point (before materialization of the data stream).

By default all message entities produced by the HTTP layer automatically carry the limit that is defined in the
application's `max-content-length` config setting. If the entity is transformed in a way that changes the
content-length and then another limit is applied then this new limit will be evaluated against the new
content-length. If the entity is transformed in a way that changes the content-length and no new limit is applied
then the previous limit will be applied against the previous content-length.
Generally this behavior should be in line with your expectations.

> <a id="1" href="#^1">[1]</a> *akka.http.parsing.max-content-length* (applying to server- as well as client-side),
*akka.http.server.parsing.max-content-length* (server-side only),
*akka.http.client.parsing.max-content-length* (client-side only) or
*akka.http.host-connection-pool.client.parsing.max-content-length* (only host-connection-pools)

### Special processing for HEAD requests

[RFC 7230](https://tools.ietf.org/html/rfc7230#section-3.3.3) defines very clear rules for the entity length of HTTP messages.

Especially this rule requires special treatment in Akka HTTP:

>
Any response to a HEAD request and any response with a 1xx
(Informational), 204 (No Content), or 304 (Not Modified) status
code is always terminated by the first empty line after the
header fields, regardless of the header fields present in the
message, and thus cannot contain a message body.

Responses to HEAD requests introduce the complexity that *Content-Length* or *Transfer-Encoding* headers
can be present but the entity is empty. This is modeled by allowing @scala[*HttpEntity.Default*]@java[*HttpEntityDefault*] and @scala[*HttpEntity.Chunked*]@java[*HttpEntityChunked*]
to be used for HEAD responses with an empty data stream.

Also, when a HEAD response has an @scala[*HttpEntity.CloseDelimited*]@java[*HttpEntityCloseDelimited*] entity the Akka HTTP implementation will *not* close the
connection after the response has been sent. This allows the sending of HEAD responses without *Content-Length*
header across persistent HTTP connections.

<a id="header-model"></a>
## 报头模型

Akka HTTP包含最常见HTTP报头的丰富模型。解析和渲染是自动完成的，因此应用程序不需要关心报头的实际语法。未显式建模的报头表示为一个 @apidoc[RawHeader] (which is essentially a String/String name/value pair)。

看看这些如何处理报头的例子:

Scala
:   @@snip [ModelSpec.scala]($test$/scala/docs/http/scaladsl/ModelSpec.scala) { #headers }

Java
:   @@snip [ModelDocTest.java]($test$/java/docs/http/javadsl/ModelDocTest.java) { #headers }

## HTTP报头

当Akka HTTP服务器接收到HTTP请求时，它会尝试将所有的头信息解析为各自的模型类。不管成功与否，HTTP层总是将所有接收到的头信息传递给应用程序。未知的标头和语法无效的标头(根据标头解析器)一样将作为 @apidoc[RawHeader] 实例变为可用。对于那些显示解析错误，将根据`illegal-header-warnings`配置的设置值记录一条警告消息。

一些报头在HTTP中有特殊的地位，因此将以不同于"常规"报头的方式对待：

Content-Type
: HTTP消息的内容类型被建模为 @apidoc[HttpEntity] 的`contentType`字段。因此，`Content-Type`标头不会出现在消息的`headers`序列中。另外，一个显式添加到请求或响应的`headers`中的`Content-Type` header实例将不会呈现在网络上，而是触发一个正在记录的警告!

Transfer-Encoding
: 带有`Transfer-Encoding: chunked`的消息 @scala[通过`HttpEntity.Chunked`]表示 @java[为一个`HttpEntityChunked`]实体。因此，块消息没有另一种更深的嵌套传输编码将不会在他们的`headers` @scala[序列]@java[列表]中有一个`Transfer-Encoding`报头。类似地，显式添加到请求或响应的`headers`中的`Transfer-Encoding`header实例将不会呈现在网络上，而是触发一个正在记录的警告!

Content-Length
: 消息的内容长度，通过它的[HttpEntity](#httpentity)建模。因此，没有`Content-Length`报头将不会成为消息的`header`序列的一部分。同样地，显式添加到请求或响应的`headers`中的`Content-Length`header实例将不会呈现在网络上，而是触发一个正在记录的警告!

Server
: `Server`报头通常自动添加到任何响应，其值可以通过`akka.http.server.server-header`设置。此外，应用程序可以用自定义报头覆盖已配置的报头，通过将其添加到响应的`header`序列中。

User-Agent
: `User-Agent`报头通常自动添加到任何请求中，其值可以通过`akka.http.client.user-agent-header`设置。此外，应用程序可以用自定义报头覆盖已配置的报头，通过将其添加到请求的`header`序列中，。

Date
: @apidoc[Date]响应头是自动添加的，但是可以通过手动提供来覆盖它。

Connection
: 在服务器端，Akka HTTP会观察是否显式添加了`Connection: close`响应报头，从而满足应用程序在发出相应响应后关闭连接的潜在需求。决定是否关闭连接的实际逻辑非常复杂。它考虑了请求的方法、协议和潜在的 @apidoc[Connection]报头，以及响应的协议、实体和潜在的 @apidoc[Connection]报头。

查看 @github[这个测试](/akka-http-core/src/test/scala/akka/http/impl/engine/rendering/ResponseRendererSpec.scala) { #connection-header-table } 对于一个完整的表。

Strict-Transport-Security
: HTTP严格传输安全(HSTS)是一种web安全策略机制，它通过`Strict-Transport-Security`报头进行通信。HSTS可以修复的最重要的安全漏洞是SSL-strippingman-in-the-middle攻击。SSL-stripping攻击的工作原理是将安全的HTTPS连接透明地转换为普通的HTTP连接。用户可以看到连接是不安全的，但关键是没有方法知道连接是否应该是安全的。HSTS通过通知浏览器到站点的连接应该始终使用TLS/SSL来解决这个问题。另请参阅 [RFC 6797](https://tools.ietf.org/html/rfc6797)。


<a id="custom-headers"></a>
## 自定义报头

有时，您可能需要对不属于HTTP的自定义头类型建模，但仍然像内置类型一样能够尽可能方便地使用它。

因为与头信息交互的方式多种多样(即试图@scala(匹配)@java[convert]一个 @apidoc[CustomHeader] @scala[反对]@java[to]一个 @apidoc[RawHeader]或其他方式等)，Akka HTTP提供了自定义标题类型的一个辅助 @scala[特质]@java(类) @scala[和它们的伴生类]。

感谢扩展 @apidoc[ModeledCustomHeader]，而不是简单的 @apidoc[CustomHeader] @scala[这样的header可以匹配]@java[以下方法由您使用]:

Scala
:   @@snip [ModeledCustomHeaderSpec.scala]($akka-http$/akka-http-tests/src/test/scala/akka/http/scaladsl/server/ModeledCustomHeaderSpec.scala) { #modeled-api-key-custom-header }

Java
:   @@snip [CustomHeaderExampleTest.java]($test$/java/docs/http/javadsl/CustomHeaderExampleTest.java) { #modeled-api-key-custom-header }

这使得这个 @scala[CustomHeader]@java[modeled custom header] 可以在以下场景中使用:

Scala
:   @@snip [ModeledCustomHeaderSpec.scala]($akka-http$/akka-http-tests/src/test/scala/akka/http/scaladsl/server/ModeledCustomHeaderSpec.scala) { #matching-examples }

Java
:   @@snip [CustomHeaderExampleTest.java]($test$/java/docs/http/javadsl/CustomHeaderExampleTest.java) { #conversion-creation-custom-header }

包括在头指令内的用法，如以下 @ref[headerValuePF](../routing-dsl/directives/header-directives/headerValuePF.md) 例子:

Scala
:   @@snip [ModeledCustomHeaderSpec.scala]($akka-http$/akka-http-tests/src/test/scala/akka/http/scaladsl/server/ModeledCustomHeaderSpec.scala) { #matching-in-routes }

Java
:   @@snip [CustomHeaderExampleTest.java]($test$/java/docs/http/javadsl/CustomHeaderExampleTest.java) { #header-value-pf }

@@@ note { .group-scala }
定义自定义报头时，最好扩展 @apidoc[ModeledCustomHeader]，而不是其父类 @apidoc[CustomHeader]。
扩展 @apidoc[ModeledCustomHeader] 的自定义报头自动符合通常应用于内置类型的模式匹配语义(例如，在Akka HTTP应用程序的路由层中将自定义报头与 @apidoc[RawHeader] 匹配)。
@@@

@@@ note { .group-java }
Implement @apidoc[ModeledCustomHeader] and @java[@javadoc[ModeledCustomHeaderFactory](akka.http.javadsl.model.headers.ModeledCustomHeaderFactory)] instead of @apidoc[CustomHeader] to be
able to use the convenience methods that allow parsing the custom user-defined header from @apidoc[HttpHeader].
@@@

## 解析/渲染

HTTP数据结构的解析和渲染经过了大量优化，对于大多数类型，目前还没有提供用于解析(或渲染)字符串或字节数组的公共API。

@@@ note
Various parsing and rendering settings are available to tweak in the configuration under `akka.http.client[.parsing]`, `akka.http.server[.parsing]` and `akka.http.host-connection-pool[.client.parsing]`, with defaults for all of these being defined in the `akka.http.parsing` configuration section.

For example, if you want to change a parsing setting for all components, you can set the `akka.http.parsing.illegal-header-warnings = off` value. However this setting can be still overridden by the more specific sections, like for example `akka.http.server.parsing.illegal-header-warnings = on`.

In this case both `client` and `host-connection-pool` APIs will see the setting `off`, however the server will see `on`.

In the case of `akka.http.host-connection-pool.client` settings, they default to settings set in `akka.http.client`, and can override them if needed. This is useful, since both `client` and `host-connection-pool` APIs,
such as the Client API @scala[`Http().outgoingConnection`]@java[`Http.get(sys).outgoingConnection`] or the Host Connection Pool APIs @scala[`Http().singleRequest`]@java[`Http.get(sys).singleRequest`] or @scala[`Http().superPool`]@java[`Http.get(sys).superPool`], usually need the same settings, however the `server` most likely has a very different set of settings.
@@@

<a id="registeringcustommediatypes"></a>
## 注册自定义媒体类型

Akka HTTP @scala[@scaladoc[预定义](akka.http.scaladsl.model.MediaTypes$)]@java[@javadoc[预定义](akka.http.javadsl.model.MediaTypes)] 最常见的媒体类型，并在解析http消息时以其类型良好的形式发出它们。有时您可能希望定义自定义媒体类型，并告知解析器基础结构如何处理这些自定义媒体类型，例如`application/custom` 被视为`NonBinary`具有`WithFixedCharset`。要实现这一点，您需要在服务器的设置中注册自定义媒体类型，通过这样配置 @apidoc[ParserSettings]：

Scala
:   @@snip [CustomMediaTypesSpec.scala]($akka-http$/akka-http-tests/src/test/scala/akka/http/scaladsl/CustomMediaTypesSpec.scala) { #application-custom }

Java
:   @@snip [CustomMediaTypesExampleTest.java]($test$/java/docs/http/javadsl/CustomMediaTypesExampleTest.java) { #application-custom-java }

你可能还想了解一下媒体类型[注册树](https://en.wikipedia.org/wiki/Media_type#Registration_trees)，以便在正确的样式/位置注册特定于供应商的媒体类型。

<a id="registeringcustomstatuscodes"></a>
## 注册自定义状态码

类似于媒体类型，Akka HTTP中众所周知的 @scala[@scaladoc:[预定义](akka.http.scaladsl.model.StatusCodes$)]@java[@javadoc:[predefines](akka.http.javadsl.model.StatusCodes)]状态码，然而，有时您可能需要使用一个自定义的(或者被迫使用返回自定义状态码的API)。与媒体类型注册类似，您可以通过配置 @apidoc[ParserSettings] 来注册自定义状态码，如下所示:

Scala
:   @@snip [CustomStatusCodesSpec.scala]($akka-http$/akka-http-tests/src/test/scala/akka/http/scaladsl/CustomStatusCodesSpec.scala) { #application-custom }

Java
:   @@snip [CustomStatusCodesExampleTest.java]($test$/java/docs/http/javadsl/CustomStatusCodesExampleTest.java) { #application-custom-java }

<a id="registeringcustommethod"></a>
## 注册自定义HTTP方法

Akka HTTP还允许您定义自定义HTTP方法，除了Akka HTTP中众所周知的 @scala[@scaladoc[预定义](akka.http.scaladsl.model.HttpMethods$)]@java[@javadoc[predefined](akka.http.javadsl.model.HttpMethods)]方法。
要使用自定义HTTP方法，您需要定义它，然后将其添加到解析器设置，如下所示:

Scala
:   @@snip [CustomHttpMethodSpec.scala]($test$/scala/docs/http/scaladsl/server/directives/CustomHttpMethodSpec.scala) { #application-custom }

Java
:   @@snip [CustomHttpMethodsExampleTest.java]($test$/java/docs/http/javadsl/server/directives/CustomHttpMethodExamplesTest.java) { #customHttpMethod }
