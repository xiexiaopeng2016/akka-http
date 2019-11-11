# pathPrefix

@@@ div { .group-scala }

## 签名

@@signature [PathDirectives.scala]($akka-http$/akka-http/src/main/scala/akka/http/scaladsl/server/directives/PathDirectives.scala) { #pathPrefix }

@@@

## 描述

针对给定的`PathMatcher`匹配并消费一个 @apidoc[RequestContext] 的未匹配路径的前缀，可能会提取一个或多个值(取决于参数的类型)。

该指令过滤传入请求，基于它的尚未被更高层次的路由结构上的其他可能存在的`pathPrefix`或 @ref[rawPathPrefix](rawPathPrefix.md) 指令匹配的URI部分。它的一个参数通常是对一个`PathMatcher`实例求值的表达式(另请参见: @ref[路径匹配器DSL](../../path-matchers.md))。

相对于它的 @ref[rawPathPrefix](rawPathPrefix.md) 对应的`pathPrefix`自动添加一个前导斜杠到它的`PathMatcher`参数，因此你不必有一个明确的斜线开始你的匹配表达式。有关路径指令之间的比较，请查看 @ref[路径指令概述](index.md#overview-path)。

根据其`PathMatcher`参数的类型，`pathPrefix`指令从URI中提取零个或多个值。如果匹配失败，则使用 @ref[空拒绝集](../../rejections.md#empty-rejections) 拒绝请求。

@@@ note
空字符串(也称为空词或标识)是字符串连接操作的 **中立元素 **，因此它将匹配所有内容且不消费任何内容。该 @ref[路径](path.md) 提供了更严格的行为。
@@@

## 示例

Scala
:  @@snip [PathDirectivesExamplesSpec.scala]($test$/scala/docs/http/scaladsl/server/directives/PathDirectivesExamplesSpec.scala) { #pathPrefix- }

Java
:  @@snip [PathDirectivesExamplesTest.java]($test$/java/docs/http/javadsl/server/directives/PathDirectivesExamplesTest.java) { #path-prefix }
