# 路由DSL概述

Akka HTTP @ref[核心服务器API](../server-side/low-level-api.md) 提供了一个 @apidoc[Flow] 或`Function`级别接口，该接口允许应用程序通过简单地将请求映射到响应来应答传入的HTTP请求(摘自 @ref[低级别服务器端示例](../server-side/low-level-api.md#http-low-level-server-side-example))：

Scala
:  @@snip [HttpServerExampleSpec.scala]($test$/scala/docs/http/scaladsl/HttpServerExampleSpec.scala) { #low-level-server-example }

Java
:  @@snip [HttpServerExampleDocTest.java]($test$/java/docs/http/javadsl/server/HttpServerExampleDocTest.java) { #request-handler }

虽然完全可以定义一个完整的REST API服务，纯粹通过对传入HttpRequest的模式匹配 @scala[(也许在几个提取器的帮助下，在[Unfiltered](https://unfiltered.ws/))方面]。由于需要大量的语法"仪式"，对于较大的服务，这种方法变得有些笨拙。同样，它可能不利于将服务定义保持为[DRY](http://en.wikipedia.org/wiki/Don%27t_repeat_yourself)。

作为一种替代方案，Akka HTTP提供了一种灵活的DSL，用于以简洁和可读的方式将您的服务行为表示为可组合元素(称为 @ref[指令](directives/index.md))的结构。指令被组装成所谓的*路由结构*，该路由结构在其顶层可以用于创建一个处理器 @apidoc[Flow](或者，一个异步处理函数)，其可以直接提供给一个`bind`调用。

@scala[从 @scaladoc[Route](akka.http.scaladsl.server.index#Route=akka.http.scaladsl.server.RequestContext=%3Escala.concurrent.Future[akka.http.scaladsl.server.RouteResult])@java[@apidoc[Route]]到flow的转换既可以使用`Route.handlerFlow`显式调用，另外，这个转换也可以由`RouteResult.route2HandlerFlow` <a id="^1" href="#1">[1]</a>隐式提供。]

下面是使用可组合高级API重写的完整示例:

Scala
:   @@snip [HttpServerExampleSpec.scala]($test$/scala/docs/http/scaladsl/HttpServerExampleSpec.scala) { #high-level-server-example }

Java
:   @@snip [HighLevelServerExample.java]($test$/java/docs/http/javadsl/server/HighLevelServerExample.java) { #high-level-server-example }

路由DSL的核心功能通过一个导入就可以使用：

Scala
:   ```scala
import akka.http.scaladsl.server.Directives._
```

Java
:   ```java
import static akka.http.javadsl.server.Directives.*;
```

@@@ div { .group-java }

Or by extending the `akka.http.javadsl.server.AllDirectives` class which brings together all directives into a single class
for easier access:

```java
extends AllDirectives
```

Of course it is possible to directly import only the directives you need (i.e. @apidoc[WebSocketDirectives] etc).

@@@

@@@ div { .group-scala }

此示例还依赖Scala XML的预定义支持，包括：

```scala
import akka.http.scaladsl.marshallers.xml.ScalaXmlSupport._
```

@@@

这里展示的非常短的示例，当然不是说明路由DSL的最佳示例。这个 @ref[更长的例子](index.md#更长的例子)可以在这方面做得更好。

为了学习如何使用路由DSL，您应该首先了解 @ref[路由](routes.md)的概念。

@@@ div { .group-scala }

> <a id="1" href="#^1">[1]</a> To be picked up automatically, the implicit conversion needs to be provided in the companion object of the source type. However, as @scala[@scaladoc[Route](akka.http.scaladsl.server.index#Route=akka.http.scaladsl.server.RequestContext=%3Escala.concurrent.Future[akka.http.scaladsl.server.RouteResult])]@java[@apidoc[Route]] is just a type alias for `RequestContext => Future[RouteResult]`, there's no
companion object for @scala[@scaladoc[Route](akka.http.scaladsl.server.index#Route=akka.http.scaladsl.server.RequestContext=%3Escala.concurrent.Future[akka.http.scaladsl.server.RouteResult])]@java[@apidoc[Route]]. Fortunately, the [implicit scope](https://www.scala-lang.org/files/archive/spec/2.11/07-implicits.html#implicit-parameters) for finding an implicit conversion also
includes all types that are "associated with any part" of the source type which in this case means that the
implicit conversion will also be picked up from `RouteResult.route2HandlerFlow` automatically.

@@@
