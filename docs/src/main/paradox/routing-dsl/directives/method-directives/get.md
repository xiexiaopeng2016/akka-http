# get

匹配带有`GET`HTTP方法的请求。

@@@ div { .group-scala }

## 签名

@@signature [MethodDirectives.scala]($akka-http$/akka-http/src/main/scala/akka/http/scaladsl/server/directives/MethodDirectives.scala) { #get }

@@@

## 描述

这个指令通过它的HTTP方法过滤传入的请求。只有带有`GET`方法的请求才会被传递到内部路由。其他所有方法均被@apidoc[MethodRejection]拒绝，它们将被默认的@ref[RejectionHandler](../../rejections.md#the-rejectionhandler)转换为一个`405 Method Not Allowed`响应。

## 示例

Scala
:  @@snip [MethodDirectivesExamplesSpec.scala]($test$/scala/docs/http/scaladsl/server/directives/MethodDirectivesExamplesSpec.scala) { #get-method }

Java
:  @@snip [MethodDirectivesExamplesTest.java]($test$/java/docs/http/javadsl/server/directives/MethodDirectivesExamplesTest.java) { #get }
