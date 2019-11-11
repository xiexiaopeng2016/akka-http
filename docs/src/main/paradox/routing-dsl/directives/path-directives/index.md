# 路径指令

@@toc { depth=1 }

@@@ index

* [path](path.md)
* [pathEnd](pathEnd.md)
* [pathEndOrSingleSlash](pathEndOrSingleSlash.md)
* [pathPrefix](pathPrefix.md)
* [pathPrefixTest](pathPrefixTest.md)
* [pathSingleSlash](pathSingleSlash.md)
* [pathSuffix](pathSuffix.md)
* [pathSuffixTest](pathSuffixTest.md)
* [rawPathPrefix](rawPathPrefix.md)
* [rawPathPrefixTest](rawPathPrefixTest.md)
* [redirectToNoTrailingSlashIfPresent](redirectToNoTrailingSlashIfPresent.md)
* [redirectToTrailingSlashIfMissing](redirectToTrailingSlashIfMissing.md)
* [ignoreTrailingSlash](ignoreTrailingSlash.md)
* [../../path-matchers](../../path-matchers.md)

@@@

<a id="overview-path"></a>
## 路径指令概述

这是一些最常见的路径指令的简短概述：

* `rawPathPrefix(x)`: 它匹配x，并且不匹配后缀(如果有)。
* `pathPrefix(x)`: 等同于 @scala[`rawPathPrefix(Slash ~ x)`]@java[`rawPathPrefix(slash().concat(segment(x)))`]。它匹配一个前导斜杠后跟 _x_ ，并且不匹配后缀。
* `path(x)`: 等同于 @scala[`rawPathPrefix(Slash ~ x ~ PathEnd)`]@java[`rawPathPrefix(slash().concat(segment(x)).concat(pathEnd()))`]。它匹配一个前导斜杠，后跟 _x_，然后是结尾。
* `pathEnd`: 等同于 @scala[`rawPathPrefix(PathEnd)`]@java[`rawPathPrefix(pathEnd())`]。仅当路径上没有剩余要匹配的内容时它才匹配。此指令不应在根路径使用，因为最小路径是单斜杠。
* `pathSingleSlash`: 等同于 @scala[`rawPathPrefix(Slash ~ PathEnd)`]@java[`rawPathPrefix(slash().concat(pathEnd()))`]。当剩余路径仅是一个斜杠时，它将匹配。
* `pathEndOrSingleSlash`: 等同于 @scala[`rawPathPrefix(PathEnd)`]@java[`rawPathPrefix(pathEnd())`] 或 @scala[`rawPathPrefix(Slash ~ PathEnd)`]@java[`rawPathPrefix(slash().concat(pathEnd()))`]。当没有剩余路径或只有一个斜杠时，它匹配。
