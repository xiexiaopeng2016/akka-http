# 4. 服务器API

除了 @ref[HTTP客户端](../client-side/index.md) Akka HTTP还提供了一种嵌入式，基于[反应-流](https://www.reactive-streams.org/)，完全异步HTTP/1.1服务器，其实现在@scala[@extref[流](akka-docs:scala/stream/index.html)]@java[@extref[流](akka-docs:java/stream/index.html)]之上。

它支持以下功能:

 * 完全支持 [HTTP persistent connections](https://en.wikipedia.org/wiki/HTTP_persistent_connection)
 * 完全支持 [HTTP pipelining](https://en.wikipedia.org/wiki/HTTP_pipelining)
 * 完全支持异步HTTP流，包括通过惯用API访问的“分块”传输编码
 * 可选的SSL/TLS加密
 * WebSocket 支持

Akka HTTP的服务器端组件分为两个层次：

@ref[核心服务器API](low-level-api.md)
:  `akka-http-core`模块内的基本的低级别服务器实现。

@ref[高级别服务器端API](../routing-dsl/index.md)
:  `akka-http`模块中的高级功能提供了非常灵活的"路由DSL"，用于优雅地定义RESTful Web服务以及典型Web服务器或框架的功能，例如URI的解构，内容协商或静态内容服务。

根据您的需求，您可以直接使用低级API或依靠高级 @ref[路由DSL](../routing-dsl/index.md)，这可以使更复杂的服务逻辑的定义变得更加容易。您还可以同时与不同的API级别进行交互，并且无论您选择哪种API级别，Akka HTTP都可以为一个或多个不同的客户端提供数千个并发连接。

@@@ note

建议阅读 @ref[Implications of the streaming nature of Request/Response Entities](../implications-of-streaming-http-entity.md) 章节，因为它解释了底层的全堆栈流概念，当来自非"流优先" HTTP服务器的背景时，这可能是出乎意料的。

@@@

@@toc { depth=3 }

@@@ index

* [low-level-api](low-level-api.md)
* [routing-dsl/index](../routing-dsl/index.md)
* [websocket-support](websocket-support.md)
* [server-https-support](server-https-support.md)
* [graceful-termination](graceful-termination.md)
* [http2](http2.md)

@@@
