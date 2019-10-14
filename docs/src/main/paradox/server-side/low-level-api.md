# 核心服务器API

核心服务器API的范围明确地集中在HTTP/1.1服务器的基本功能上：

 * 连接管理
 * 解析和渲染消息和报头
 * 超时管理(用于请求和连接)
 * 响应顺序(用于透明流管道支持)

典型HTTP服务器的所有非核心功能(例如请求路由，文件服务，压缩等)都留在了 @ref[更高的层级](../routing-dsl/index.md) 上，它们不是由`akka-http-core`-级别服务器本身实现的。除了一般的关注点之外，这种设计还使服务器核心保持小巧轻便，易于理解和维护。

@@@ note

建议阅读 @ref[Implications of the streaming nature of Request/Response Entities](../implications-of-streaming-http-entity.md) 章节，因为它解释了底层的全堆栈流概念，当来自非"流优先" HTTP服务器的背景时，这可能是出乎意料的。

@@@

## 流和HTTP

Akka HTTP服务器是在 @scala[@extref[Streams](akka-docs:scala/stream/index.html)]@java[@extref[Streams](akka-docs:java/stream/index.html)] 之上实现的，并大量使用它 - 在它的实现中，以及在它的API的所有级别上。

在连接级别，Akka HTTP提供了与 @scala[@extref[使用流IO](akka-docs:scala/stream/stream-io.html)]@java[@extref[使用流IO](akka-docs:java/stream/stream-io.html)] 基本上相同的接口：套接字绑定表示为传入连接的流。应用程序从流的源中提取连接，并为每个连接提供一个 @apidoc[Flow[HttpRequest, HttpResponse, \_]]，以将请求"转化"为响应。

除了将绑定在服务器端的套接字视为 @apidoc[Source[IncomingConnection, \_]]，并将每个连接视为 @apidoc[Source[HttpRequest, \_]] 及其 @apidoc[Sink[HttpResponse, \_]]，流抽象还存在于单个HTTP消息中： HTTP请求和响应的实体通常建模为 @apidoc[Source[ByteString, \_]]。另请参阅 @ref[HTTP模型](../common/http-model.md)，以获取有关如何在Akka HTTP中表示HTTP消息的更多信息。

## 开始和结束

在最基本的级别上，通过调用 @scala[@scaladoc[akka.http.scaladsl.Http](akka.http.scaladsl.Http$)]@java[@javadoc[akka.http.javadsl.Http](akka.http.javadsl.Http)]扩展的`bind`方法来绑定Akka HTTP服务器：

Scala
:   @@snip [HttpServerExampleSpec.scala]($test$/scala/docs/http/scaladsl/HttpServerExampleSpec.scala) { #binding-example }

Java
:   @@snip [HttpServerExampleDocTest.java]($test$/java/docs/http/javadsl/server/HttpServerExampleDocTest.java) { #binding-example }

`Http().bind`方法的参数指定要绑定到的接口和端口，并在处理传入HTTP连接期间注册兴趣(interest)。此外，该方法还允许定义套接字选项以及根据您的需要配置服务器的大量设置。

`bind`方法的结果是一个 @apidoc[Source[Http.IncomingConnection]]，其必须由应用程序消耗，以接受传入的连接。在将这个source实现为处理管道的一部分之前，不会执行实际的绑定。万一绑定失败(例如，因为端口已经忙)，则实现的流将立即终止，并带有相应的异常。当传入连接source的订阅者取消其订阅时，绑定将被释放(即底层套接字未绑定)。或者，也可以使用`Http.ServerBinding`实例的`unbind()`方法，该方法是作为连接源实现过程的一部分创建的。`Http.ServerBinding`还提供了一种方法来获取绑定套接字的实际本地地址，这在例如绑定到端口零时非常有用(从而让OS选择可用端口)。

<a id="http-low-level-server-side-example"></a>
## 请求-响应周期

接受新连接后，它将被发布为一个`Http.IncomingConnection`，其中包含远程地址和方法，以提供一个 @apidoc[Flow[HttpRequest, HttpResponse, \_]] 处理通过此连接传入的请求。

通过使用处理程序调用一个`handleWithXXX`方法来处理请求，该处理程序可以是

>
 * 一个 @apidoc[Flow[HttpRequest, HttpResponse, \_]] 对于 `handleWith`,
 * 一个 @scala[`HttpRequest => HttpResponse`]@java[`Function<HttpRequest, HttpResponse>`] 函数对于 `handleWithSyncHandler`,
 * 一个 @scala[`HttpRequest => Future[HttpResponse]`]@java[`Function<HttpRequest, CompletionStage<HttpResponse>>`] 函数对于 `handleWithAsyncHandler`。

这是一个完整的例子：

Scala
:   @@snip [HttpServerExampleSpec.scala]($test$/scala/docs/http/scaladsl/HttpServerExampleSpec.scala) { #full-server-example }

Java
:   @@snip [HttpServerExampleDocTest.java]($test$/java/docs/http/javadsl/server/HttpServerExampleDocTest.java) { #full-server-example }

在此示例中，请求是通过转换请求流来处理的，通过一个函数 @scala[`HttpRequest => HttpResponse`]@java[`Function<HttpRequest, HttpResponse>`]，使用`handleWithSyncHandler`(或等价的，Akka 流的`map`操作)。根据用例，可以使用Akka Stream的组合器来提供请求处理程序的许多其他方法。如果应用程序提供@apidoc[Flow]，则应用程序还有责任为每个请求生成一个准确的响应，并且响应的顺序与关联请求的顺序相匹配(这是有重大关系的，如果启用了HTTP管道处理多个传入请求的处理可能会重叠)。当依靠`handleWithSyncHandler`或`handleWithAsyncHandler`，`map`或`mapAsync`流运算符时，将自动满足此要求。

有关创建请求处理程序的更方便的高级DSL，请参见 @ref[路由DSL概述](../routing-dsl/overview.md)。

### 流请求/响应实体

通过 @apidoc[HttpEntity] 的子类支持HTTP消息实体的流。应用程序在接收请求时需要处理流实体，在许多情况下，在构造响应时也需要处理流实体。有关替代方法的描述，请参见 @ref[HttpEntity](../common/http-model.md#httpentity)。

如果您依赖Akka HTTP提供的 @ref[编组](../common/marshalling.md) 和/或 @ref[解组](../common/unmarshalling.md) 功能，则转到自定义类型和从流实体转出的转换会非常方便。

<a id="http-closing-connection-low-level"></a>
### 关闭连接

当处理流取消其上游订阅或对对方关闭连接时，HTTP连接将关闭。另一种更方便的方法是显式地向 @apidoc[HttpResponse] 添加一个`Connection: close` 报头。这个响应将是连接上的最后一个响应，服务器将在发送连接时主动关闭连接。

如果请求实体已被取消，例如通过将其附加到`Sink.cancelled()`或仅部分消耗(如，通过使用`take`组合器)，连接也将被关闭。为了防止这种行为，实体应该通过将它附加到`Sink.ignore()`来显式地耗尽。

## 配置服务器端HTTPS

有关在服务器端配置和使用HTTPS的详细文档，请参阅 @ref[服务器端HTTPS支持](server-https-support.md)。

<a id="http-server-layer"></a>
## 单独的HTTP层用法

由于其基于响应流的性质，Akka HTTP层与底层TCP接口是完全可分离的。尽管在大多数应用程序中，此"特征"并不是至关重要的，但在某些情况下，能够针对并非来自网络而是来自某些网络的数据“运行” HTTP层（以及可能的更高层）可能会很有用。其他来源。这可能有用的潜在方案包括测试，调试或低级事件源（例如，通过重播网络流量）。

it can be useful in certain cases to be able to "run" the HTTP layer (and, potentially, higher-layers) against data that do not come from the network but rather some other source. Potential scenarios where this might be useful include tests, debugging or low-level event-sourcing (e.g by replaying network traffic).

@@@ div { .group-scala }
在服务器端，独立HTTP层形成一个 @apidoc[BidiFlow]，其定义如下：

@@snip [Http.scala]($akka-http$/akka-http-core/src/main/scala/akka/http/scaladsl/Http.scala) { #server-layer }

你可以通过调用`Http().serverLayer`方法的两个重载之一来创建`Http.ServerLayer`的实例，这也允许不同程度的配置。
@@@
@@@ div { .group-java }
On the server-side the stand-alone HTTP layer forms a @apidoc[BidiFlow[HttpResponse, SslTlsOutbound, SslTlsInbound, HttpRequest, NotUsed]],
that is a stage that "upgrades" a potentially encrypted raw connection to the HTTP level.

You create an instance of the layer by calling one of the two overloads of the `Http.get(system).serverLayer` method,
which also allows for varying degrees of configuration. Note, that the returned instance is not reusable and can only
be materialized once.
@@@

## 控制服务器并行性

Request handling can be parallelized on two axes, by handling several connections in parallel and by
relying on HTTP pipelining to send several requests on one connection without waiting for a response first. In both
cases the client controls the number of ongoing requests. To prevent being overloaded by too many requests, Akka HTTP
can limit the number of requests it handles in parallel.

To limit the number of simultaneously open connections, use the `akka.http.server.max-connections` setting. This setting
applies to all of `Http.bindAndHandle*` methods. If you use `Http.bind`, incoming connections are represented by
a @apidoc[Source[IncomingConnection, ...]]. Use Akka Stream's combinators to apply backpressure to control the flow of
incoming connections, e.g. by using `throttle` or `mapAsync`.

HTTP pipelining is generally discouraged (and [disabled by most browsers](https://en.wikipedia.org/w/index.php?title=HTTP_pipelining&oldid=700966692#Implementation_in_web_browsers)) but
is nevertheless fully supported in Akka HTTP. The limit is applied on two levels. First, there's the
`akka.http.server.pipelining-limit` config setting which prevents that more than the given number of outstanding requests
is ever given to the user-supplied handler-flow. On the other hand, the handler flow itself can apply any kind of throttling
itself. If you use the `Http.bindAndHandleAsync`
entry-point, you can specify the `parallelism` argument (which defaults to `1`, which means that pipelining is disabled) to control the
number of concurrent requests per connection. If you use `Http.bindAndHandle` or `Http.bind`, the user-supplied handler
flow has full control over how many request it accepts simultaneously by applying backpressure. In this case, you can
e.g. use Akka Stream's `mapAsync` combinator with a given parallelism to limit the number of concurrently handled requests.
Effectively, the more constraining one of these two measures, config setting and manual flow shaping, will determine
how parallel requests on one connection are handled.

<a id="handling-http-server-failures-low-level"></a>
## Handling HTTP Server failures in the Low-Level API

There are various situations when failure may occur while initialising or running an Akka HTTP server.
Akka by default will log all these failures, however sometimes one may want to react to failures in addition to them
just being logged, for example by shutting down the actor system, or notifying some external monitoring end-point explicitly.

There are multiple things that can fail when creating and materializing an HTTP Server (similarly, the same applied to
a plain streaming `Tcp()` server). The types of failures that can happen on different layers of the stack, starting
from being unable to start the server, and ending with failing to unmarshal an HttpRequest, examples of failures include
(from outer-most, to inner-most):

 * Failure to `bind` to the specified address/port,
 * Failure while accepting new `IncomingConnection`s, for example when the OS has run out of file descriptors or memory,
 * Failure while handling a connection, for example if the incoming @apidoc[HttpRequest] is malformed.

This section describes how to handle each failure situation, and in which situations these failures may occur.

#### Bind failures

The first type of failure is when the server is unable to bind to the given port. For example when the port
is already taken by another application, or if the port is privileged (i.e. only usable by `root`).
In this case the "binding future" will fail immediately, and we can react to it by listening on the @scala[Future's]@java[CompletionStage’s] completion:

Scala
:   @@snip [HttpServerExampleSpec.scala]($test$/scala/docs/http/scaladsl/HttpServerExampleSpec.scala) { #binding-failure-handling }

Java
:   @@snip [HttpServerExampleDocTest.java]($test$/java/docs/http/javadsl/server/HttpServerExampleDocTest.java) { #binding-failure-handling }

Once the server has successfully bound to a port, the @apidoc[Source[IncomingConnection, \_]] starts running and emitting
new incoming connections. This source technically can signal a failure as well, however this should only happen in very
dramatic situations such as running out of file descriptors or memory available to the system, such that it's not able
to accept a new incoming connection. Handling failures in Akka Streams is pretty straight forward, as failures are signaled
through the stream starting from the stage which failed, all the way downstream to the final stages.

#### Connections Source failures

In the example below we add a custom @apidoc[GraphStage] (see @scala[@extref[Custom stream processing](akka-docs:scala/stream/stream-customize.html)]@java[@extref[Custom stream processing](akka-docs:java/stream/stream-customize.html)]) in order to react to the
stream's failure. We signal a `failureMonitor` actor with the cause why the stream is going down, and let the Actor
handle the rest – maybe it'll decide to restart the server or shutdown the ActorSystem, that however is not our concern anymore.

Scala
:   @@snip [HttpServerExampleSpec.scala]($test$/scala/docs/http/scaladsl/HttpServerExampleSpec.scala) { #incoming-connections-source-failure-handling }

Java
:   @@snip [HttpServerExampleDocTest.java]($test$/java/docs/http/javadsl/server/HttpServerExampleDocTest.java) { #incoming-connections-source-failure-handling }

#### Connection failures

The third type of failure that can occur is when the connection has been properly established,
however afterwards is terminated abruptly – for example by the client aborting the underlying TCP connection.

To handle this failure we can use the same pattern as in the previous snippet, however apply it to the connection's Flow:

Scala
:   @@snip [HttpServerExampleSpec.scala]($test$/scala/docs/http/scaladsl/HttpServerExampleSpec.scala) { #connection-stream-failure-handling }

Java
:   @@snip [HttpServerExampleDocTest.java]($test$/java/docs/http/javadsl/server/HttpServerExampleDocTest.java) { #connection-stream-failure-handling }


Note that this is when the TCP connection is closed correctly, if the client just goes away, for example because of
a network failure, it will not be seen as this kind of stream failure. It will instead be detected through the
@ref[idle timeout](../common/timeouts.md#超时)).


These failures can be described more or less infrastructure related, they are failing bindings or connections.
Most of the time you won't need to dive into those very deeply, as Akka will simply log errors of this kind
anyway, which is a reasonable default for such problems.

In order to learn more about handling exceptions in the actual routing layer, which is where your application code
comes into the picture, refer to @ref[Exception Handling](../routing-dsl/exception-handling.md) which focuses explicitly on explaining how exceptions
thrown in routes can be handled and transformed into @apidoc[HttpResponse] s with appropriate error codes and human-readable failure descriptions.
