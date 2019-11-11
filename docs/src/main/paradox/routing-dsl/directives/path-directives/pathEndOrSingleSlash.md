# pathEndOrSingleSlash

@@@ div { .group-scala }

## 签名

@@signature [PathDirectives.scala]($akka-http$/akka-http/src/main/scala/akka/http/scaladsl/server/directives/PathDirectives.scala) { #pathEndOrSingleSlash }

@@@

## 描述

仅当 @apidoc[RequestContext] 的未匹配路径为空或仅包含一个斜杠时，才将请求传递到其内部路由。

该指令是`rawPathPrefix(Slash.? ~ PathEnd)`的简单别名，主要用于内部级别，以将"已完全匹配的路径"与其他替代项区分开(请参见下面的示例)。有关路径指令之间的比较，请查看 @ref[路径指令概述](index.md#overview-path)。

它等同于 `pathEnd | pathSingleSlash`，但效率稍微高一点。

## 示例

Scala
:  @@snip [PathDirectivesExamplesSpec.scala]($test$/scala/docs/http/scaladsl/server/directives/PathDirectivesExamplesSpec.scala) { #pathEndOrSingleSlash- }

Java
:  @@snip [PathDirectivesExamplesTest.java]($test$/java/docs/http/javadsl/server/directives/PathDirectivesExamplesTest.java) { #path-end-or-single-slash }
