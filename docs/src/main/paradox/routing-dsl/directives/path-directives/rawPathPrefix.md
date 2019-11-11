# rawPathPrefix

@@@ div { .group-scala }

## 签名

@@signature [PathDirectives.scala]($akka-http$/akka-http/src/main/scala/akka/http/scaladsl/server/directives/PathDirectives.scala) { #rawPathPrefix }

@@@

## 描述

针对给定的`PathMatcher`匹配并消费一个 @apidoc[RequestContext] 的未匹配路径的前缀，可能会提取一个或多个值(取决于参数的类型)。

该指令过滤传入请求，基于它的尚未被更高层次的路由结构上的其他可能存在的`rawPathPrefix`或 @ref[pathPrefix](pathPrefix.md) 指令匹配的URI部分。它的一个参数通常是对一个`PathMatcher`实例求值的表达式(另请参见: @ref[路径匹配器DSL](../../path-matchers.md))。

与 @ref[pathPrefix](pathPrefix.md) 相反，对应的`rawPathPrefix`并 *不* 会自动添加一个前导斜杠到其`PathMatcher`参数。而是将其`PathMatcher`参数按原样应用于未匹配的路径。有关路径指令之间的比较，请查看 @ref[路径指令概述](index.md#overview-path)。

根据其`PathMatcher`参数的类型，`rawPathPrefix`指令从URI中提取零个或多个值。如果匹配失败，则使用 @ref[空拒绝集](../../rejections.md#empty-rejections) 拒绝请求。

## 示例

Scala
:  @@snip [PathDirectivesExamplesSpec.scala]($test$/scala/docs/http/scaladsl/server/directives/PathDirectivesExamplesSpec.scala) { #completeWithUnmatchedPath #rawPathPrefix- }

Java
:  @@snip [PathDirectivesExamplesTest.java]($test$/java/docs/http/javadsl/server/directives/PathDirectivesExamplesTest.java) { #raw-path-prefix-test }
