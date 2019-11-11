# post

使用HTTP方法`POST`匹配请求。

@@@ div { .group-scala }

## 签名

@@signature [MethodDirectives.scala]($akka-http$/akka-http/src/main/scala/akka/http/scaladsl/server/directives/MethodDirectives.scala) { #post }

@@@

## 描述

该指令通过其HTTP方法过滤传入的请求。只有带有`POST`方法的请求才会传递到内部路由。所有其他均被 @apidoc[MethodRejection] 拒绝，默认的 @ref[RejectionHandler](../../rejections.md#the-rejectionhandler)将其转换为`405 Method Not Allowed`响应。

## 示例

Scala
:  @@snip [MethodDirectivesExamplesSpec.scala]($test$/scala/docs/http/scaladsl/server/directives/MethodDirectivesExamplesSpec.scala) { #post-method }

Java
:  @@snip [MethodDirectivesExamplesTest.java]($test$/java/docs/http/javadsl/server/directives/MethodDirectivesExamplesTest.java) { #post }
