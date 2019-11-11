# 完成(complete)

@@@ div { .group-scala }
## 签名

```scala
def complete[T :ToResponseMarshaller](value: T): StandardRoute
def complete(response: HttpResponse): StandardRoute
def complete(status: StatusCode): StandardRoute
def complete[T :Marshaller](status: StatusCode, value: T): StandardRoute
def complete[T :Marshaller](status: Int, value: T): StandardRoute
def complete[T :Marshaller](status: StatusCode, headers: Seq[HttpHeader], value: T): StandardRoute
def complete[T :Marshaller](status: Int, headers: Seq[HttpHeader], value: T): StandardRoute
```

所示的签名已简化，真正的签名使用了磁铁(magnet)。<a id="^1" href="#1">[1]</a>

> <a id="1" href="#^1">[1]</a> 查看 [磁铁模式](http://spray.io/blog/2012-12-13-the-magnet-pattern/) 解释基于磁铁的重载.

@@@

## 描述

使用给定的参数完成请求。

`complete`使用给定的参数来构建一个 @scala[@scaladoc[路由](akka.http.scaladsl.server.index#Route=akka.http.scaladsl.server.RequestContext=%3Escala.concurrent.Future[akka.http.scaladsl.server.RouteResult])]@java[@apidoc[Route]]，它简单的调用 @apidoc[RequestContext] 的`complete`，使用各自的 @apidoc[HttpResponse] 实例。完成请求将向响应发送“备份”路由结构，在该结构中所有运行的逻辑都表明包装指令可能已链接到 @ref[RouteResult](../../routes.md#routeresult) 未来的转换链中。

@java[Please note that the `complete` directive has multiple variants, like the ones shown in the examples.]

## 示例

Scala
:  @@snip [RouteDirectivesExamplesSpec.scala]($test$/scala/docs/http/scaladsl/server/directives/RouteDirectivesExamplesSpec.scala) { #complete-examples }

Java
:  @@snip [RouteDirectivesExamplesTest.java]($test$/java/docs/http/javadsl/server/directives/RouteDirectivesExamplesTest.java) { #complete }
