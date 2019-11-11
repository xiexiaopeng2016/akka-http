# pathSuffix

@@@ div { .group-scala }

## 签名

@@signature [PathDirectives.scala]($akka-http$/akka-http/src/main/scala/akka/http/scaladsl/server/directives/PathDirectives.scala) { #pathSuffix }

@@@

## 描述

针对给定的`PathMatcher`匹配并消费 @apidoc[RequestContext] 的未匹配路径的一个后缀，可能会提取一个或多个值(取决于参数的类型)。

该指令过滤传入请求，基于尚未被更高级别路由结构上其他可能存在的路径匹配指令匹配的URI部分。它的一个参数通常是对一个`PathMatcher`实例求值的表达式(另请参见: @ref[路径匹配器DSL](../../path-matchers.md))。

与 @ref[pathPrefix](pathPrefix.md) 相反，这个指令从右边(即结尾)匹配并消费未匹配的路径。

@@@ warning { title="Caution" }
出于效率方面的考虑，给定`PathMatcher`必须以相反的段顺序匹配所需的后缀，即`pathSuffix("baz" / "bar")`将会匹配`/foo/bar/baz`！段匹配中的顺序不会颠倒。
@@@

根据其`PathMatcher`参数的类型，`pathSuffix`指令从URI中提取零个或多个值。如果匹配失败，则使用 @ref[空拒绝集](../../rejections.md#empty-rejections) 拒绝请求。

## 示例

Scala
:  @@snip [PathDirectivesExamplesSpec.scala]($test$/scala/docs/http/scaladsl/server/directives/PathDirectivesExamplesSpec.scala) { #completeWithUnmatchedPath #pathSuffix- }

Java
:  @@snip [PathDirectivesExamplesTest.java]($test$/java/docs/http/javadsl/server/directives/PathDirectivesExamplesTest.java) { #path-suffix }
