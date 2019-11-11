# path

@@@ div { .group-scala }

## 签名

@@signature [PathDirectives.scala]($akka-http$/akka-http/src/main/scala/akka/http/scaladsl/server/directives/PathDirectives.scala) { #path }

@@@

## 描述

针对给定的`PathMatcher`匹配 @apidoc[RequestContext] 的完全未匹配路径，可能会提取一个或多个值(取决于参数的类型)。

这个指令过滤传入的请求，基于它们的URI中尚未被其他可能存在的 @ref[pathPrefix](pathPrefix.md) 指令匹配的部分，这些指令位于更高级别的路由结构。它的一个参数通常是一个对`PathMatcher`实例求值的表达式(另请参见: @ref[路径匹配器DSL](../../path-matchers.md))。

与 @ref[rawPathPrefix](rawPathPrefix.md) 或 @ref[rawPathPrefixTest](rawPathPrefixTest.md) 指令相反，`path`会自动在其`PathMatcher`参数中添加一个前导斜杠，因此您不必以明确的斜杠开始匹配的表达式。

该`path`指令尝试匹配 **完整** 的剩余路径，而不仅仅是前缀。如果只想匹配路径前缀，然后将进一步的筛选委托给路由结构中的较低级别，请使用 @ref[pathPrefix](pathPrefix.md) 指令代替。因此，将一个`path`或 @ref[pathPrefix](pathPrefix.md)指令嵌套在另一个`path`指令底下是没有意义的，因为它们不可能匹配(因为`path`指令下的不匹配路径始终为空)。有关路径指令之间的比较，请查看 @ref[路径指令概述](index.md#overview-path)。

根据它的`PathMatcher`参数的类型，`path`指令从URI中提取零个或多个值。如果匹配失败，则使用 @ref[空拒绝集](../../rejections.md#empty-rejections) 拒绝请求。

@@@ note
空字符串(也称为空词或标识)是字符串连接操作的 **中立元素**，因此它将匹配所有内容，但请记住，`path`要求匹配其余全部路径，因此(`/`)将成功，而(`/whatever`)将失败。 @ref[pathPrefix](pathPrefix.md) 提供更自由的行为。
@@@

## 示例

Scala
:  @@snip [PathDirectivesExamplesSpec.scala]($test$/scala/docs/http/scaladsl/server/directives/PathDirectivesExamplesSpec.scala) { #path-example }

Java
:  @@snip [PathDirectivesExamplesTest.java]($test$/java/docs/http/javadsl/server/directives/PathDirectivesExamplesTest.java) { #path-dsl }
