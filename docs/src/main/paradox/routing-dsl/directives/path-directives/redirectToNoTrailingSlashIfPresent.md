# redirectToNoTrailingSlashIfPresent

@@@ div { .group-scala }

## 签名

@@signature [PathDirectives.scala]($akka-http$/akka-http/src/main/scala/akka/http/scaladsl/server/directives/PathDirectives.scala) { #redirectToNoTrailingSlashIfPresent }

@@@

## 描述

如果请求的路径的确以结尾`/`字符结尾，重定向到不包含结尾斜杠的相同路径。

重定向HTTP客户端到相同的资源，但不带结尾的`/`，以防请求包含它。

当使用给定的重定向响应代码（即MovedPermanently或TemporaryRedirect等）重定向HttpResponse 以及包含“ 单击我跟随重定向 ”链接的简单HTML页面时，以防客户端无法访问或出于安全原因拒绝使用，自动跟随重定向。

When redirecting an HttpResponse with the given redirect response code (i.e. `MovedPermanently` or `TemporaryRedirect` etc.) as well as a simple HTML page containing a "*click me to follow redirect*" link to be used in case the client can not, or refuses to for security reasons, automatically follow redirects.

Please note that the inner paths **MUST NOT** end with an explicit trailing slash (e.g. `"things"./`)
for the re-directed-to route to match.

A good read on the subject of how to deal with trailing slashes is available on [Google Webmaster Central - To Slash or not to Slash](https://webmasters.googleblog.com/2010/04/to-slash-or-not-to-slash.html).

See also @ref[redirectToTrailingSlashIfMissing](redirectToTrailingSlashIfMissing.md) which achieves the opposite - redirecting paths in case they do *not* have a trailing slash.

## Example

Scala
:  @@snip [PathDirectivesExamplesSpec.scala]($test$/scala/docs/http/scaladsl/server/directives/PathDirectivesExamplesSpec.scala) { #redirectToNoTrailingSlashIfPresent-0 }

Java
:  @@snip [PathDirectivesExamplesTest.java]($test$/java/docs/http/javadsl/server/directives/PathDirectivesExamplesTest.java) { #redirect-notrailing-slash-present }
