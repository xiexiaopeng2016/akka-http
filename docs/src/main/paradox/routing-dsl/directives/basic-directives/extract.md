# 提取(extract)

@@@ div { .group-scala }

## 签名

@@signature [BasicDirectives.scala]($akka-http$/akka-http/src/main/scala/akka/http/scaladsl/server/directives/BasicDirectives.scala) { #extract }

@@@

## 描述

`extract`指令用作 @ref[自定义指令](../custom-directives.md) 的构造块，用于从 @apidoc[RequestContext] 提取数据并将其提供给内部路由。这是一种比提取一个值更常见的特殊情况， @ref[textract](textract.md)指令可用于提取多个值。

有关类似指令的概述，请参见为 @ref[为内部路由提供值](index.md#providedirectives)。

## 示例

Scala
:  @@snip [BasicDirectivesExamplesSpec.scala]($test$/scala/docs/http/scaladsl/server/directives/BasicDirectivesExamplesSpec.scala) { #extract0 }

Java
:  @@snip [BasicDirectivesExamplesTest.java]($test$/java/docs/http/javadsl/server/directives/BasicDirectivesExamplesTest.java) { #extract }
