# 服务器-发送事件支持

服务器-发送事件(SSE)是一种轻量级的[标准化](https://www.w3.org/TR/eventsource)协议，用于将通知从HTTP服务器推送到客户端。与提供双向通信的WebSocket相比，SSE仅允许从服务器到客户端的单向通信。如果您仅需这些，SSE的优点是更加简单，仅依赖HTTP并为浏览器断开的连接提供重试语义。

根据SSE规范，客户端可以通过HTTP从服务器请求事件流。服务器以媒体类型`text/event-stream`进行响应，它具有固定的字符编码UTF-8，并保持响应打开，以便在可用时将事件发送给客户端。事件是带有字段并以空行终止的文本结构，例如

```
data: { "username": "John Doe" }
event: added
id: 42

data: another event
```

客户端通过 @scala[`Last-Event-ID`]@java[`LastEventId`] 标头可选择地将最后一次看到的事件发送给服务器，例如在重新连接之后。

## 模型

Akka HTTP将事件流表示为 @apidoc[Source[ServerSentEvent, \_]]，其中 @apidoc[ServerSentEvent] 是具有以下只读属性的 @scala[样例] 类：

- @scala[`data: String`]@java[`String data`] – 实际有效载荷，可能跨越多行
- @scala[`eventType: Option[String]`]@java[`Optional<String> type`] – 可选的限定词，例如“添加”，“删除”等。
- @scala[`id: Option[String]`]@java[`Optional<String> id`] – 可选标识符
- @scala[`retry: Option[Int]`]@java[`OptionalInt retry`] – 可选的重新连接延迟，以毫秒为单位

根据SSE规范，Akka HTTP还提供了 @scala[`Last-Event-ID`]@java[`LastEventId`] 标头和 @scala[`text/event-stream`]@java[`TEXT_EVENT_STREAM`] 媒体类型。

## 服务器端用法：编组

为了用事件流响应HTTP请求，您必须将 @apidoc[EventStreamMarshalling] 定义的`ToResponseMarshaller[Source[ServerSentEvent, \_]]`隐式引入定义相应路由的作用域内：

Scala
:  @@snip [ServerSentEventsExampleSpec.scala]($test$/scala/docs/http/scaladsl/ServerSentEventsExampleSpec.scala) { #event-stream-marshalling-example }

Java
:  @@snip [EventStreamMarshallingTest.java]($akka-http$/akka-http-tests/src/test/java/akka/http/javadsl/marshalling/sse/EventStreamMarshallingTest.java) { #event-stream-marshalling-example }

## 客户端用法：解组

为了将事件流解组为 @apidoc[Source[ServerSentEvent, \_]]，你必须将 @apidoc[EventStreamUnmarshalling] 定义的`FromEntityUnmarshaller[Source[ServerSentEvent, _]]`隐式引入作用域：

Scala
:  @@snip [ServerSentEventsExampleSpec.scala]($test$/scala/docs/http/scaladsl/ServerSentEventsExampleSpec.scala) { #event-stream-unmarshalling-example }

Java
:  @@snip [EventStreamMarshallingTest.java]($akka-http$/akka-http-tests/src/test/java/akka/http/javadsl/unmarshalling/sse/EventStreamUnmarshallingTest.java) { #event-stream-unmarshalling-example }

请注意，如果您正在寻找一种永久订阅事件流的弹性方式，则Alpakka提供了[EventSource](https://developer.lightbend.com/docs/alpakka/current/sse.html)连接器，它会自动与上次看到的事件的ID重新连接。
