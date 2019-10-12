# HTTP模型

Akka HTTP模型包含一个结构深刻的，完全不变的，基于样例类的模型，该模型包含所有主要的HTTP数据结构，例如HTTP请求，响应和公共报头。它位于*akka-http-core*模块中，并构成了大多数Akka HTTP API的基础。

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

Streaming entity types (i.e. all but @scala[`Strict`]@java[`HttpEntityStrict`]) cannot be shared or serialized. To create a strict, shareable copy of an entity or message use `HttpEntity.toStrict` or `HttpMessage.toStrict` which returns a @scala[`Future`]@java[`CompletionStage`] of the object with the body data collected into a @apidoc[akka.util.ByteString].

The @scala[@apidoc[HttpEntity] companion object]@java[class `HttpEntities`] contains @scala[several helper constructors]@java[static methods] to create entities from common types easily.

You can @scala[pattern match over]@java[use] the @scala[subtypes]@java[`isX` methods] of @apidoc[HttpEntity] @java[to find out of which subclass an entity is] if you want to provide
special handling for each of the subtypes. However, in many cases a recipient of an @apidoc[HttpEntity] doesn't care about
of which subtype an entity is (and how data is transported exactly on the HTTP layer). Therefore, the general method
@scala[`HttpEntity.dataBytes`]@java[`HttpEntity.getDataBytes()`] is provided which returns a @apidoc[Source[ByteString, \_]] that allows access to the data of an
entity regardless of its concrete subtype.

@@@ note { title='When to use which subtype?' }

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
## Header Model

Akka HTTP contains a rich model of the most common HTTP headers. Parsing and rendering is done automatically so that
applications don't need to care for the actual syntax of headers. Headers not modelled explicitly are represented
as a @apidoc[RawHeader] (which is essentially a String/String name/value pair).

See these examples of how to deal with headers:

Scala
:   @@snip [ModelSpec.scala]($test$/scala/docs/http/scaladsl/ModelSpec.scala) { #headers }

Java
:   @@snip [ModelDocTest.java]($test$/java/docs/http/javadsl/ModelDocTest.java) { #headers }

## HTTP Headers

When the Akka HTTP server receives an HTTP request it tries to parse all its headers into their respective
model classes. Independently of whether this succeeds or not, the HTTP layer will
always pass on all received headers to the application. Unknown headers as well as ones with invalid syntax (according
to the header parser) will be made available as @apidoc[RawHeader] instances. For the ones exhibiting parsing errors a
warning message is logged depending on the value of the `illegal-header-warnings` config setting.

Some headers have special status in HTTP and are therefore treated differently from "regular" headers:

Content-Type
: The Content-Type of an HTTP message is modeled as the `contentType` field of the @apidoc[HttpEntity].
The `Content-Type` header therefore doesn't appear in the `headers` sequence of a message.
Also, a `Content-Type` header instance that is explicitly added to the `headers` of a request or response will
not be rendered onto the wire and trigger a warning being logged instead!

Transfer-Encoding
: Messages with `Transfer-Encoding: chunked` are represented @scala[via the `HttpEntity.Chunked`]@java[as a `HttpEntityChunked`] entity.
As such chunked messages that do not have another deeper nested transfer encoding will not have a `Transfer-Encoding`
header in their `headers` @scala[sequence]@java[list].
Similarly, a `Transfer-Encoding` header instance that is explicitly added to the `headers` of a request or
response will not be rendered onto the wire and trigger a warning being logged instead!

Content-Length
: The content length of a message is modelled via its [HttpEntity](#httpentity). As such no `Content-Length` header will ever
be part of a message's `header` sequence.
Similarly, a `Content-Length` header instance that is explicitly added to the `headers` of a request or
response will not be rendered onto the wire and trigger a warning being logged instead!

Server
: A `Server` header is usually added automatically to any response and its value can be configured via the
`akka.http.server.server-header` setting. Additionally an application can override the configured header with a
custom one by adding it to the response's `header` sequence.

User-Agent
: A `User-Agent` header is usually added automatically to any request and its value can be configured via the
`akka.http.client.user-agent-header` setting. Additionally an application can override the configured header with a
custom one by adding it to the request's `header` sequence.

Date
: The @apidoc[Date] response header is added automatically but can be overridden by supplying it manually.

Connection
: On the server-side Akka HTTP watches for explicitly added `Connection: close` response headers and as such honors
the potential wish of the application to close the connection after the respective response has been sent out.
The actual logic for determining whether to close the connection is quite involved. It takes into account the
request's method, protocol and potential @apidoc[Connection] header as well as the response's protocol, entity and
potential @apidoc[Connection] header. See @github[this test](/akka-http-core/src/test/scala/akka/http/impl/engine/rendering/ResponseRendererSpec.scala) { #connection-header-table } for a full table of what happens when.

Strict-Transport-Security
: HTTP Strict Transport Security (HSTS) is a web security policy mechanism which is communicated by the
`Strict-Transport-Security` header. The most important security vulnerability that HSTS can fix is SSL-stripping
man-in-the-middle attacks. The SSL-stripping attack works by transparently converting a secure HTTPS connection into a
plain HTTP connection. The user can see that the connection is insecure, but crucially there is no way of knowing
whether the connection should be secure. HSTS addresses this problem by informing the browser that connections to the
site should always use TLS/SSL. See also [RFC 6797](https://tools.ietf.org/html/rfc6797).


<a id="custom-headers"></a>
## Custom Headers

Sometimes you may need to model a custom header type which is not part of HTTP and still be able to use it
as convenient as is possible with the built-in types.

Because of the number of ways one may interact with headers (i.e. try to @scala[match]@java[convert] a @apidoc[CustomHeader] @scala[against]@java[to] a @apidoc[RawHeader]
or the other way around etc), a helper @scala[trait]@java[classes] for custom Header types @scala[and their companions classes ]are provided by Akka HTTP.
Thanks to extending @apidoc[ModeledCustomHeader] instead of the plain @apidoc[CustomHeader] @scala[such header can be matched]@java[the following methods are at your disposal]:

Scala
:   @@snip [ModeledCustomHeaderSpec.scala]($akka-http$/akka-http-tests/src/test/scala/akka/http/scaladsl/server/ModeledCustomHeaderSpec.scala) { #modeled-api-key-custom-header }

Java
:   @@snip [CustomHeaderExampleTest.java]($test$/java/docs/http/javadsl/CustomHeaderExampleTest.java) { #modeled-api-key-custom-header }

Which allows this @scala[CustomHeader]@java[modeled custom header] to be used in the following scenarios:

Scala
:   @@snip [ModeledCustomHeaderSpec.scala]($akka-http$/akka-http-tests/src/test/scala/akka/http/scaladsl/server/ModeledCustomHeaderSpec.scala) { #matching-examples }

Java
:   @@snip [CustomHeaderExampleTest.java]($test$/java/docs/http/javadsl/CustomHeaderExampleTest.java) { #conversion-creation-custom-header }

Including usage within the header directives like in the following @ref[headerValuePF](../routing-dsl/directives/header-directives/headerValuePF.md) example:

Scala
:   @@snip [ModeledCustomHeaderSpec.scala]($akka-http$/akka-http-tests/src/test/scala/akka/http/scaladsl/server/ModeledCustomHeaderSpec.scala) { #matching-in-routes }

Java
:   @@snip [CustomHeaderExampleTest.java]($test$/java/docs/http/javadsl/CustomHeaderExampleTest.java) { #header-value-pf }

@@@ note { .group-scala }
When defining custom headers, it is better to extend @apidoc[ModeledCustomHeader] instead of its parent @apidoc[CustomHeader].
Custom headers that extend @apidoc[ModeledCustomHeader] automatically comply with the pattern matching semantics that usually apply to built-in
types (such as matching a custom header against a @apidoc[RawHeader] in routing layers of Akka HTTP applications).
@@@

@@@ note { .group-java }
Implement @apidoc[ModeledCustomHeader] and @java[@javadoc[ModeledCustomHeaderFactory](akka.http.javadsl.model.headers.ModeledCustomHeaderFactory)] instead of @apidoc[CustomHeader] to be
able to use the convenience methods that allow parsing the custom user-defined header from @apidoc[HttpHeader].
@@@

## Parsing / Rendering

Parsing and rendering of HTTP data structures is heavily optimized and for most types there's currently no public API
provided to parse (or render to) Strings or byte arrays.

@@@ note
Various parsing and rendering settings are available to tweak in the configuration under `akka.http.client[.parsing]`,
`akka.http.server[.parsing]` and `akka.http.host-connection-pool[.client.parsing]`, with defaults for all of these
being defined in the `akka.http.parsing` configuration section.

For example, if you want to change a parsing setting for all components, you can set the `akka.http.parsing.illegal-header-warnings = off`
value. However this setting can be still overridden by the more specific sections, like for example `akka.http.server.parsing.illegal-header-warnings = on`.

In this case both `client` and `host-connection-pool` APIs will see the setting `off`, however the server will see `on`.

In the case of `akka.http.host-connection-pool.client` settings, they default to settings set in `akka.http.client`,
and can override them if needed. This is useful, since both `client` and `host-connection-pool` APIs,
such as the Client API @scala[`Http().outgoingConnection`]@java[`Http.get(sys).outgoingConnection`] or the Host Connection Pool APIs @scala[`Http().singleRequest`]@java[`Http.get(sys).singleRequest`]
or @scala[`Http().superPool`]@java[`Http.get(sys).superPool`], usually need the same settings, however the `server` most likely has a very different set of settings.
@@@

<a id="registeringcustommediatypes"></a>
## Registering Custom Media Types

Akka HTTP @scala[@scaladoc[predefines](akka.http.scaladsl.model.MediaTypes$)]@java[@javadoc[predefines](akka.http.javadsl.model.MediaTypes)] most commonly encountered media types and emits them in their well-typed form while parsing http messages.
Sometimes you may want to define a custom media type and inform the parser infrastructure about how to handle these custom
media types, e.g. that `application/custom` is to be treated as `NonBinary` with `WithFixedCharset`. To achieve this you
need to register the custom media type in the server's settings by configuring @apidoc[ParserSettings] like this:

Scala
:   @@snip [CustomMediaTypesSpec.scala]($akka-http$/akka-http-tests/src/test/scala/akka/http/scaladsl/CustomMediaTypesSpec.scala) { #application-custom }

Java
:   @@snip [CustomMediaTypesExampleTest.java]($test$/java/docs/http/javadsl/CustomMediaTypesExampleTest.java) { #application-custom-java }

You may also want to read about MediaType [Registration trees](https://en.wikipedia.org/wiki/Media_type#Registration_trees), in order to register your vendor specific media types
in the right style / place.

<a id="registeringcustomstatuscodes"></a>
## Registering Custom Status Codes

Similarly to media types, Akka HTTP @scala[@scaladoc:[predefines](akka.http.scaladsl.model.StatusCodes$)]@java[@javadoc:[predefines](akka.http.javadsl.model.StatusCodes)]
well-known status codes, however sometimes you may need to use a custom one (or are forced to use an API which returns custom status codes).
Similarly to the media types registration, you can register custom status codes by configuring @apidoc[ParserSettings] like this:

Scala
:   @@snip [CustomStatusCodesSpec.scala]($akka-http$/akka-http-tests/src/test/scala/akka/http/scaladsl/CustomStatusCodesSpec.scala) { #application-custom }

Java
:   @@snip [CustomStatusCodesExampleTest.java]($test$/java/docs/http/javadsl/CustomStatusCodesExampleTest.java) { #application-custom-java }

<a id="registeringcustommethod"></a>
## Registering Custom HTTP Method

Akka HTTP also allows you to define custom HTTP methods, other than the well-known methods @scala[@scaladoc[predefined](akka.http.scaladsl.model.HttpMethods$)]@java[@javadoc[predefined](akka.http.javadsl.model.HttpMethods)] in Akka HTTP.
To use a custom HTTP method, you need to define it, and then add it to parser settings like below:

Scala
:   @@snip [CustomHttpMethodSpec.scala]($test$/scala/docs/http/scaladsl/server/directives/CustomHttpMethodSpec.scala) { #application-custom }

Java
:   @@snip [CustomHttpMethodsExampleTest.java]($test$/java/docs/http/javadsl/server/directives/CustomHttpMethodExamplesTest.java) { #customHttpMethod }
