# rawPathPrefixTest

@@@ div { .group-scala }

## 签名

@@signature [PathDirectives.scala]($akka-http$/akka-http/src/main/scala/akka/http/scaladsl/server/directives/PathDirectives.scala) { #rawPathPrefixTest }

@@@

## 描述

检查 @apidoc[RequestContext] 的未匹配路径是否具有与给定`PathMatcher`匹配的前缀。可能会提取一个或多个值(取决于参数的类型)，但不消费未匹配路径中的匹配项。

此指令与 @ref[pathPrefix](pathPrefix.md) 指令非常相似，唯一的区别是它匹配的路径前缀(如果匹配)*不*会被消费。因此，即使在指令成功匹配并将请求传递到其内部路由的情况下，也仍然保留了 @apidoc[RequestContext] 的未匹配路径。

有关如何创建一个`PathMatcher`的更多信息，请参见 @ref[路径匹配器DSL](../../path-matchers.md)。

与 @ref[pathPrefixTest](pathPrefixTest.md) 相反，对应的`rawPathPrefixTest` *不* 会自动添加一个前导斜杠到其`PathMatcher`参数。而是将其`PathMatcher`参数按原样应用于未匹配的路径。

根据它的`PathMatcher`参数的类型，`rawPathPrefixTest`指令从URI中提取零个或多个值。如果匹配失败，则使用 @ref[空拒绝集](../../rejections.md#empty-rejections) 拒绝请求。

## 示例

Scala
:  @@snip [PathDirectivesExamplesSpec.scala]($test$/scala/docs/http/scaladsl/server/directives/PathDirectivesExamplesSpec.scala) { #completeWithUnmatchedPath #rawPathPrefixTest- }

Java
:  @@snip [PathDirectivesExamplesTest.java]($test$/java/docs/http/javadsl/server/directives/PathDirectivesExamplesTest.java) { #raw-path-prefix-test }
