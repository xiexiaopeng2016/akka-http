# pathEnd

@@@ div { .group-scala }

## 签名

@@signature [PathDirectives.scala]($akka-http$/akka-http/src/main/scala/akka/http/scaladsl/server/directives/PathDirectives.scala) { #pathEnd }

@@@

## 描述

仅当 @apidoc[RequestContext] 的不匹配路径为空时，即请求路径已由更高级别的 @ref[路径](path.md) 或 @ref[pathPrefix](pathPrefix.md) 指令完全匹配，才将请求传递到其内部路由。

该指令是`rawPathPrefix(PathEnd)`的简单别名，主要用于内部级别，以将"已完全匹配的路径"与其他替代方法区分开(请参见下面的示例)。有关路径指令之间的比较，请查看 @ref[路径指令概述](index.md#overview-path)。

## 示例

Scala
:  @@snip [PathDirectivesExamplesSpec.scala]($test$/scala/docs/http/scaladsl/server/directives/PathDirectivesExamplesSpec.scala) { #pathEnd- }

Java
:  @@snip [PathDirectivesExamplesTest.java]($test$/java/docs/http/javadsl/server/directives/PathDirectivesExamplesTest.java) { #path-end }
