# 路由DSL

除了 @ref[核心服务器API](../server-side/low-level-api.md)外，Akka HTTP还提供了非常灵活的“路由DSL”，用于优雅地定义RESTful Web服务。它可以弥补低级API的不足之处，并提供典型Web服务器或框架的许多高级功能，例如URI的解构，内容协商或静态内容服务。

@@@ note
建议阅读 @ref[请求/响应实体的流性质的含义](../implications-of-streaming-http-entity.md)部分，因为它解释了底层的全堆栈流概念，当来自非“流优先” HTTP服务器的背景时，这可能是意想不到的。
from a background with non-"streaming first" HTTP Servers.
@@@

@@toc { depth=1 }

@@@ index

* [overview](overview.md)
* [routes](routes.md)
* [directives/index](directives/index.md)
* [rejections](rejections.md)
* [exception-handling](exception-handling.md)
* [path-matchers](path-matchers.md)
* [case-class-extraction](case-class-extraction.md)
* [source-streaming-support](source-streaming-support.md)
* [testkit](testkit.md)
* [http-app](HttpApp.md)

@@@

## 最小的例子

这是一个依赖路由DSL的完整的，非常基础的Akka HTTP应用程序：

Scala
:  @@snip [HttpServerExampleSpec.scala]($test$/scala/docs/http/scaladsl/HttpServerExampleSpec.scala) { #minimal-routing-example }

Java
:  @@snip [HttpServerMinimalExampleTest.java]($test$/java/docs/http/javadsl/HttpServerMinimalExampleTest.java) { #minimal-routing-example }

它在本地主机上启动HTTP Server，并通过简单的响应回复`/hello`GET请求。

@@@ warning { title="API可能会改变" }
以下示例使用实验性功能，其API在将来的Akka HTTP版本中会发生变化。有关此标记的更多信息，请参见Akka文档中的 @extref:[@DoNotInherit和@ApiMayChange标记](akka-docs:common/binary-compatibility-rules.html#the-donotinherit-and-apimaychange-markers)。
@@@

为了帮助启动服务器，Akka HTTP提供了一个名为 @apidoc[HttpApp]的实验性帮助程序类。这与使用 @apidoc[HttpApp]重写之前的示例相同：

Scala
:  @@snip [HttpAppExampleSpec.scala]($test$/scala/docs/http/scaladsl/HttpAppExampleSpec.scala) { #minimal-routing-example }

Java
:  @@snip [HttpAppExampleTest.java]($test$/java/docs/http/javadsl/server/HttpAppExampleTest.java) { #minimal-routing-example }

有关使用此方法设置服务器的更多详细信息，请参见 @ref[HttpApp Bootstrap](HttpApp.md)。

@@@ div { .group-scala }

## 更长的例子

以下是一个Akka HTTP路由定义，试图展示一些功能。产生的服务实际上并没有做任何有用的事情，但是其定义应该使您对路由DSL的实际API定义有一个感觉：

@@snip [HttpServerExampleSpec.scala]($test$/scala/docs/http/scaladsl/HttpServerExampleSpec.scala) { #long-routing-example }

@@@

## 与Akka Typed的互动

自从Akka版本开始`2.5.22`，Akka typed已可以投入生产，但是Akka HTTP仍在使用untyped `ActorSystem`。下面的示例将演示如何在同一应用程序中一起使用Akka HTTP和Akka Typed。

我们将创建一个小型Web服务器，该服务器负责记录构建作业及其状态和持续时间，按ID和状态查询作业，以及清除作业历史记录。

首先让我们开始定义`Behavior`，它将充当构建作业信息的存储库：

Scala
:  @@snip [HttpServerWithTypedSpec.scala]($test$/scala-2.12+/docs/http/scaladsl/HttpServerWithTypedSpec.scala) { #akka-typed-behavior }

现在，让我们定义JSON编组器和解组器：

Scala
:  @@snip [HttpServerWithTypedSpec.scala]($test$/scala-2.12+/docs/http/scaladsl/HttpServerWithTypedSpec.scala) { #akka-typed-json }


下一步是定义 @apidoc[Route$]，它将与前面定义的`Behavior`通信并处理所有可能的响应

Scala
:  @@snip [HttpServerWithTypedSpec.scala]($test$/scala-2.12+/docs/http/scaladsl/HttpServerWithTypedSpec.scala) { #akka-typed-route }


最后，我们只需要引导我们的Web服务器并实例化我们的`Behavior`：

Scala
:  @@snip [HttpServerWithTypedSpec.scala]($test$/scala-2.12+/docs/http/scaladsl/HttpServerWithTypedSpec.scala) { #akka-typed-bootstrap }


请注意，`akka.actor.typed.ActorSystem`用`toClassic`转换，它来自`import akka.actor.typed.scaladsl.adapter._`。如果您使用的是Akka 2.5.x，则此转换方法命名为`toUntyped`。

## 动态路由示例

在为每个请求评估路由时，可以在运行时进行更改。请注意，每个访问都可能在单独的线程上进行，因此任何共享的可变状态都必须是线程安全的。

以下是Akka HTTP路由定义，它允许在运行时动态添加新的或更新模拟端点以及相关的请求-响应对(pairs)。

Scala
:  @@snip [HttpServerExampleSpec.scala]($test$/scala/docs/http/scaladsl/HttpServerExampleSpec.scala) { #dynamic-routing-example }

Java
:  @@snip [HttpServerDynamicRoutingExampleTest.java]($test$/java/docs/http/javadsl/HttpServerDynamicRoutingExampleTest.java) { #dynamic-routing-example }

例如，假设我们对body执行一个POST请求：

```json
{
  "path": "test",
  "requests": [
    {"id": 1},
    {"id": 2}
  ],
  "responses": [
    {"amount": 1000},
    {"amount": 2000}
  ]
}
```

随后发送带`{"id": 1}`的请求给`/test`，将响应`{"amount": 1000}`。

## 在高级API中处理HTTP服务器故障

在多种情况下，初始化或运行Akka HTTP服务器时可能会发生故障。默认情况下，Akka将记录所有这些故障，但是有时除了记录故障之外，还可能希望对故障做出反应，例如，通过关闭actor系统或明确通知某些外部监视端点。

### 绑定失败

例如，服务器可能无法绑定到给定的端口。例如，当该端口已被另一个应用程序占用时，或者该端口具有特权(即，仅可用于`root`)。在这种情况下，"binding future"将立即失败，我们可以通过侦听@scala[`Future`]@java[`CompletionStage`]的完成情况对此做出反应：

Scala
:  @@snip [HttpServerExampleSpec.scala]($test$/scala/docs/http/scaladsl/HttpServerExampleSpec.scala) { #binding-failure-high-level-example }

Java
:  @@snip [HighLevelServerBindFailureExample.java]($test$/java/docs/http/javadsl/server/HighLevelServerBindFailureExample.java) { #binding-failure-high-level-example }

@@@ note

有关可能发生的故障种类的更底层概述，以及对它们的更精细控制，请参阅 @ref[使用低级别API处理HTTP Server故障](../server-side/low-level-api.md#handling-http-server-failures-low-level)文档。
@@@

### 路由DSL中的故障和异常

路由DSL中的异常处理是通过提供@apidoc[ExceptionHandler]来完成的，在文档的 @ref[异常处理](exception-handling.md)部分中对此进行了详细介绍。您可以使用它们将异常转换为带有适当的错误代码和可读的故障描述的@apidoc[HttpResponse]。

## 文件上传

有关处理上传的高级指令，请参见 @ref[FileUploadDirectives](directives/file-upload-directives/index.md)。

可以通过接受*Multipart.FormData*实体来处理简单文件上传，例如带有*文件*输入的浏览器表单。请注意，主体部分是*Source*，而不是所有部分都立即可用，单个的主体部分有效负载也是如此，因此您需要为文件和表单字段使用这些流。

这是一个简单的示例，仅将上载的文件转储到磁盘上的临时文件中，收集一些表单字段并将条目保存到虚拟数据库中：

Scala
:  @@snip [FileUploadExamplesSpec.scala]($test$/scala/docs/http/scaladsl/server/FileUploadExamplesSpec.scala) { #simple-upload }

Java
:  @@snip [FileUploadExamplesTest.java]($test$/java/docs/http/javadsl/server/FileUploadExamplesTest.java) { #simple-upload }


您可以在上传的文件到达时对其进行转换，而不是像前面的示例一样将它们存储在临时文件中。在此示例中，我们接受任意数量的`.csv`文件，将它们解析为几行，将每一行拆分之后将其发送给actor进行进一步处理：

Scala
:  @@snip [FileUploadExamplesSpec.scala]($test$/scala/docs/http/scaladsl/server/FileUploadExamplesSpec.scala) { #stream-csv-upload }

Java
:  @@snip [FileUploadExamplesTest.java]($test$/java/docs/http/javadsl/server/FileUploadExamplesTest.java) { #stream-csv-upload }

## 配置服务器端HTTPS

有关在服务器端配置和使用HTTPS的详细文档，请参阅 @ref[服务器端HTTPS支持](../server-side/server-https-support.md)。
