<a id="routes"></a>
# 路由

"路由"是Akka HTTP路由DSL的中心概念。您使用DSL构建的所有结构，无论是由单行还是跨越几百行构成，都是将一个 @apidoc[RequestContext] 转化成一个 @scala[`Future[RouteResult]`]@java[`CompletionStage<RouteResult>`]的 @scala[`type`]@java[`function`]。

@@@ div { .group-scala }

```scala
type Route = RequestContext => Future[RouteResult]
```
它是一个将 @apidoc[RequestContext]转化成`Future[RouteResult]`的函数的简单别名。

@@@

@@@ div { .group-java }

A @scala[@scaladoc[Route](akka.http.scaladsl.server.index#Route=akka.http.scaladsl.server.RequestContext=%3Escala.concurrent.Future[akka.http.scaladsl.server.RouteResult])]@java[@apidoc[Route]] itself is a function that operates on a @apidoc[RequestContext] and returns a @apidoc[RouteResult]. The
@apidoc[RequestContext] is a data structure that contains the current request and auxiliary data like the so far unmatched
path of the request URI that gets passed through the route structure. It also contains the current `ExecutionContext`
and `akka.stream.Materializer`, so that these don't have to be passed around manually.

@@@

通常，当路由接收到一个请求(或更确切地说是 @apidoc[RequestContext])时，它可以执行一个以下操作：

 * 通过返回`requestContext.complete(...)`完成请求
 * 通过返回`requestContext.reject(...)`拒绝请求(查看 @ref[Rejections](rejections.md#rejections)) 
 * 通过返回`requestContext.fail(...)`失败请求，或只是抛出一个异常(查看 @ref[Exception Handling](exception-handling.md#exception-handling))
 * 做任何类型的异步处理并即时返回一个被稍后完成的 @scala[`Future[RouteResult]`]@java[`CompletionStage<RouteResult>`]

第一种情况很清楚，通过调用`complete`，一个给定响应将作为对请求的回应发送给客户端。在第二种情况下，"拒绝"表示路由不想处理该请求。在关于路由组合的后面章节中，您将看到它的好处。

通过使用`Route.seal`， @scala[@scaladoc[路由](akka.http.scaladsl.server.index#Route=akka.http.scaladsl.server.RequestContext=%3Escala.concurrent.Future[akka.http.scaladsl.server.RouteResult])]@java[@apidoc[Router]]可以被"封闭"，其依赖于在范围内的`RejectionHandler`和 @apidoc[ExceptionHandler] 实例来将拒绝和异常转换成相应的客户端HTTP响应。 @ref[封闭一个路由](#封闭一个路由)将在后面详细介绍。

使用`Route.handlerFlow`或`Route.asyncHandler`一个 @scala[@scaladoc[路由](akka.http.scaladsl.server.index#Route=akka.http.scaladsl.server.RequestContext=%3Escala.concurrent.Future[akka.http.scaladsl.server.RouteResult])]@java[@apidoc[Route]]可以被提升到一个处理器 @apidoc[Flow] 或异步处理器函数，可以与来自 @ref[核心服务器API](../server-side/low-level-api.md)的`bindAndHandleXXX`调用一起使用。

注意：那里还有一个从 @scala[@scaladoc[路由](akka.http.scaladsl.server.index#Route=akka.http.scaladsl.server.RequestContext=%3Escala.concurrent.Future[akka.http.scaladsl.server.RouteResult])]@java[@apidoc[Route]] 到 @apidoc[Flow[HttpRequest, HttpResponse, Unit]] 的隐式转换，定义在 @apidoc[RouteResult] 伴生对象，其依赖`Route.handlerFlow`。

<a id="requestcontext"></a>
## RequestContext

请求上下文包装了一个 @apidoc[HttpRequest] 实例，用附加信息来充实它，它们是路由逻辑通常需要的，例如，`ExecutionContext`，@apidoc[Materializer]，@apidoc[LoggingAdapter]和已配置的 @apidoc[RoutingSettings]。它还包含`unmatchedPath`，一个描述有多少个请求URI尚未被 @ref[Path指令](directives/path-directives/index.md#pathdirectives) 匹配的值。

@apidoc[RequestContext] 本身是不可变的，但是包含一些辅助方法，其允许方便地创建已修改的副本。

<a id="routeresult"></a>
## RouteResult

@apidoc[RouteResult] 是一种简单的代数数据类型(ADT)，可对一个@scala[@scaladoc[路由](akka.http.scaladsl.server.index#Route=akka.http.scaladsl.server.RequestContext=%3Escala.concurrent.Future[akka.http.scaladsl.server.RouteResult])]@java[@apidoc[Route]]的非错误结果进行建模。它的定义如下：

@@@ div { .group-scala }

```scala
sealed trait RouteResult

object RouteResult {
  final case class Complete(response: HttpResponse) extends RouteResult
  final case class Rejected(rejections: immutable.Seq[Rejection]) extends RouteResult
}
```

@@@

通常你不会自己创建任何 @apidoc[RouteResult] 实例，而是依赖于预先定义的 @ref[RouteDirectives](directives/route-directives/index.md#routedirectives)(例如 @ref[complete](directives/route-directives/complete.md#complete), @ref[reject](directives/route-directives/reject.md#reject) 或 @ref[redirect](directives/route-directives/redirect.md#redirect))，或由[RequestContext](#requestcontext)中的各自的方法代替。

## 组合路由

这里是我们需要的三个基本操作来从简单的路由构建更复杂的路由：

 * 路由转换，路由转换，其将处理委托给另一个"内部"路由，但在该过程中更改传入请求、传出响应或两者的某些属性
 * 路由过滤，它只允许满足给定筛选条件的请求通过，并拒绝所有其他请求
 * 路由链接，如果给定的第一个路由被拒绝，则尝试第二个路由

最后一点是使用连接操作符`~`实现的，这是一个扩展方法，当您`import akka.http.scaladsl.server.Directives._`时可用。前两点是由所谓的@ref[指令](directives/index.md#directives)提供的，Akka HTTP已经预先定义了大量的指令，您也可以轻松地自己创建它们。@ref[指令](directives/index.md#directives)提供了Akka HTTP的大部分功能和灵活性。

<a id="the-routing-tree"></a>
## 路由树

本质上，当您通过`concat`方法将指令和自定义路由组合起来时，您将构建一个形成树状形式的路由结构。当一个请求进入时，它被从根处注入到这个树中，并以深度优先的方式向下流经所有分支，直到某个节点完成它或完全拒绝它。

考虑以下示意图示例：

@@@ div { .group-scala }

```scala
val route =
  a {
    concat(
      b {
        concat(
          c {
            ... // route 1
          },
          d {
            ... // route 2
          },
          ... // route 3
        )
      },
      e {
        ... // route 4
      }
    )
  }
```

@@@

@@@ div { .group-java }

```java
import static akka.http.javadsl.server.Directives.*;

Route route =
  directiveA(concat(() ->
    directiveB(concat(() ->
      directiveC(
        ... // route 1
      ),
      directiveD(
        ... // route 2
      ),
      ... // route 3
    )),
    directiveE(
      ... // route 4
    )
  ));
```

@@@

这里五个指令组成一个路由树。

 * 路由1仅会到达，假如指令 `a`，`b`和`c`都让请求通过
 * 路由2将会运行，假如`a`和`b`通过, `c`拒绝，且`d`通过。
 * 路由3将会运行，假如`a`和`b`通过, 但是`c`和`d`拒绝.

因此，路由3可以被看作是一条"全覆盖"的路由，只有当连接到前面位置的线路被拒绝时，它才会开始运行。这种机制可以使复杂的过滤逻辑非常容易实现：只需将最具体的用例放在前面，将最一般的用例放在后面。

## 封闭一个路由

如 @ref[拒绝](rejections.md) 和 @ref[异常处理](exception-handling.md) 中所述，通常有两种方法来处理拒绝和异常。

 * 将拒绝/异常处理程序引入 @scala[`顶层的隐式作用域`]@java[`seal()` method of the @scala[@scaladoc[Route](akka.http.scaladsl.server.index#Route=akka.http.scaladsl.server.RequestContext=%3Escala.concurrent.Future[akka.http.scaladsl.server.RouteResult])]@java[@apidoc[Route]]]
 * 为 @ref[handleRejections](directives/execution-directives/handleRejections.md#handlerejections)和 @ref[handleExceptions](directives/execution-directives/handleExceptions.md#handleexceptions)指令提供处理器作为参数

在第一种情况下，您的处理程序将被"封闭"(这意味着它将接收默认的处理程序作为所有您的处理程序不能自己处理的情况的回退(fallback))，并用于路由结构内部本身不处理的所有拒绝/异常。

### Route.seal()方法修改HttpResponse

在应用程序代码中，与 @ref[测试代码](testkit.md#testing-sealed-routes) 不同，您无需使用`Route.seal()`方法来封闭一个路由。只要将隐式拒绝和/或异常处理程序带到顶级作用域，路由就会被封闭。

但是，您可以使用`Route.seal()`对来自路由的HttpResponse进行修改。例如，如果你想要添加特殊的报头，但仍使用默认的拒绝处理程序，则可以执行以下操作。在下面的情况中，特殊报头已添加到与路由不匹配的拒绝响应，以及与路由匹配的成功响应中。

Scala
:   @@snip [RouteSealExampleSpec.scala]($root$/src/test/scala/docs/http/scaladsl/RouteSealExampleSpec.scala) { #route-seal-example }

Java
:   @@snip [RouteSealExample.java]($root$/src/test/java/docs/http/javadsl/RouteSealExample.java) { #route-seal-example }

### 在Java和Scala DSL之间转换路由

在某些情况下，构建暴露路由的可重用库时，能够在Java和Scala DSL表示形式之间转换路由可能会很有用。您可以在Java DSL路由上使用`asScala`方法，或使用`RouteAdapter`来包装Scala DSL路由。

将Scala DSL路由转换为Java DSL：

Scala
:   @@snip [RouteJavaScalaDslConversionSpec.scala]($akka-http$//akka-http-tests/src/test/scala/akka/http/scaladsl/RouteJavaScalaDslConversionSpec.scala) { #scala-to-java }

Java
:   @@snip [RouteSealExample.java]($akka-http$/akka-http-tests/src/test/java/docs/http/javadsl/server/RouteJavaScalaDslConversionTest.java) { #scala-to-java }

将Java DSL路由转换为Scala DSL：

Scala
:   @@snip [RouteJavaScalaDslConversionSpec.scala]($akka-http$//akka-http-tests/src/test/scala/akka/http/scaladsl/RouteJavaScalaDslConversionSpec.scala) { #java-to-scala }

Java
:   @@snip [RouteSealExample.java]($akka-http$/akka-http-tests/src/test/java/docs/http/javadsl/server/RouteJavaScalaDslConversionTest.java) { #java-to-scala }
