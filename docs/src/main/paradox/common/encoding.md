# 编码/解码

[HTTP规范](https://tools.ietf.org/html/rfc7231#section-3.1.2.1)定义了一个`Content-Encoding`标头，它表示HTTP消息的实体主体是否“编码”，如果是，则由这个算法编码。唯一常用的内容编码是压缩算法。

当前，Akka HTTP支持使用`gzip`或`deflate`编码对HTTP请求和响应进行压缩和解压缩。这方面的核心逻辑位于 @scala[@scaladoc[akka.http.scaladsl.coding](akka.http.scaladsl.coding.index) 包。]@java[@javadoc[akka.http.javadsl.coding.Coder](akka.http.javadsl.coding.Coder) enum class.]

## 服务器端

该支持不会自动启用，但必须明确要求。有关使用 @ref[路由DSL](../routing-dsl/index.md) 启用消息编码/解码的信息，请参阅 @ref[CodingDirectives](../routing-dsl/directives/coding-directives/index.md)。

## 客户端

目前不存在对客户端上的响应进行解码的高级或自动支持。

下面的示例显示如何基于`Content-Encoding`标头手动解码响应：

Scala
:   @@snip [HttpClientDecodingExampleSpec.scala]($test$/scala/docs/http/scaladsl/HttpClientDecodingExampleSpec.scala) { #single-request-decoding-example }

Java
:   @@snip [HttpClientDecodingExampleTest.java]($test$/java/docs/http/javadsl/HttpClientDecodingExampleTest.java) { #single-request-decoding-example }
