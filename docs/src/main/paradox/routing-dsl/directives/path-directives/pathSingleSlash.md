# pathSingleSlash

@@@ div { .group-scala }

## 签名

@@signature [PathDirectives.scala]($akka-http$/akka-http/src/main/scala/akka/http/scaladsl/server/directives/PathDirectives.scala) { #pathSingleSlash }

@@@

## 描述

如果 @apidoc[RequestContext] 的未匹配路径仅包含一个斜杠，则仅将请求传递到其内部路由。

该指令是`pathPrefix(PathEnd)`的一个简单别名，主要用于匹配在内部级别上的根URI(`/`)请求，以将"所有匹配的路径段"与其他选项区分开来(请参见下面的示例)。有关路径指令之间的比较，请查看 @ref[路径指令概述](index.md#overview-path)。

## 示例

Scala
:  @@snip [PathDirectivesExamplesSpec.scala]($test$/scala/docs/http/scaladsl/server/directives/PathDirectivesExamplesSpec.scala) { #pathSingleSlash- }

Java
:  @@snip [PathDirectivesExamplesTest.java]($test$/java/docs/http/javadsl/server/directives/PathDirectivesExamplesTest.java) { #path-single-slash }
