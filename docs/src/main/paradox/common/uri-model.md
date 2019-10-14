## URI模型

Akka HTTP提供了自己的专用 @apidoc[Uri] 模型类，该类针对其他类型的HTTP模型中的性能和惯用用法进行了调整。例如，将 @apidoc[HttpRequest] 的目标URI解析为这种类型，其中应用了所有字符转义和其他URI特定的语义。

### 解析URI字符串

我们遵循[RFC 3986](https://tools.ietf.org/html/rfc3986#section-1.1.2)来实现URI解析规则。当您尝试解析URI字符串时，Akka HTTP会在内部创建 @apidoc[Uri] 类的实例，其中包含建模的URI组件。

例如，以下代码创建一个简单的有效URI的实例：

Scala
:   
    ```
    Uri("http://localhost")
    ```
    
Java
:   
    ```
    Uri.create("http://localhost");
    ```

下面是有效的URI字符串的一些例子，以及如何构建一个 @apidoc[Uri] 模型类的实例 @scala[，使用 `Uri.from()` 方法，通过传递 `scheme`, `host`, `path` 和 `query` 参数]

Scala
:   @@snip [UriSpec.scala]($akka-http$/akka-http-core/src/test/scala/akka/http/scaladsl/model/UriSpec.scala) { #valid-uri-examples }

Java
:   @@snip [UriTest.scala]($akka-http$/akka-http-core/src/test/java/akka/http/javadsl/model/UriTest.java) { #valid-uri-examples }

对于URI的部件的确切定义，例如 `scheme`，`path` 和 `query`，参考[RFC 3986](https://tools.ietf.org/html/rfc3986#section-1.1.2)。以下是一些概述：

```
  foo://example.com:8042/over/there?name=ferret#nose
  \_/   \______________/\_________/ \_________/ \__/
   |           |            |            |        |
scheme     authority       path        query   fragment
   |   _____________________|__
  / \ /                        \
  urn:example:animal:ferret:nose
```

对于URI中的"特殊"字符，通常使用如下所示的百分号编码。在 @ref[URI中的查询字符串](#uri中的查询字符串) 部分中将更详细地讨论百分号编码。

Scala
:   @@snip [UriSpec.scala]($akka-http$/akka-http-core/src/test/scala/akka/http/scaladsl/model/UriSpec.scala) { #dont-double-decode }

Java
:   @@snip [UriTest.java]($akka-http$/akka-http-core/src/test/java/akka/http/javadsl/model/UriTest.java) { #dont-double-decode }


#### 无效的URI字符串和IllegalUriException

当一个无效的URI字符串传递给`Uri()`时，或抛出一个`IllegalUriException`。

Scala
:   @@snip [UriSpec.scala]($akka-http$/akka-http-core/src/test/scala/akka/http/scaladsl/model/UriSpec.scala) { #illegal-cases-immediate-exception }

Java
:   @@snip [UriTest.java]($akka-http$/akka-http-core/src/test/java/akka/http/javadsl/model/UriTest.java) { #illegal-scheme #illegal-userinfo #illegal-percent-encoding #illegal-path #illegal-path-with-control-char }

#### 提取URI组件的指令

要使用指令提取URI组件，请参阅以下参考资料：

* @ref:[extractUri](../routing-dsl/directives/basic-directives/extractUri.md)
* @ref:[extractScheme](../routing-dsl/directives/scheme-directives/extractScheme.md)
* @ref:[scheme](../routing-dsl/directives/scheme-directives/scheme.md)
* @ref:[PathDirectives](../routing-dsl/directives/path-directives/index.md)
* @ref:[ParameterDirectives](../routing-dsl/directives/parameter-directives/index.md)

### 获取原始请求URI

有时可能需要获取传入URI的“原始”值，而不对其应用任何转义或解析。尽管这种用例很少见，但偶尔也会出现。可以在Akka HTTP服务器端获得“原始”请求URI，通过打开`akka.http.server.raw-request-uri-header`标记。可以在Akka HTTP Server端获取“原始”请求URI 。启用后，一个`Raw-Request-URI`报头将添加到每个请求。该报头将保存使用的原来的原始请求的URI。有关示例，请检查参考配置。

### URI中的查询字符串

尽管URI的任何部分都可以包含特殊字符，但URI中的查询字符串更常见的具有特殊字符，这些字符通常是[百分号编码](https://en.wikipedia.org/wiki/Percent-encoding)的。

@scala[@apidoc[Uri]类的`query()`方法]@java[The method `Uri::query()`]返回URI的查询字符串，它是在`Query`类的实例中建模的。
当你通过传递URI字符串实例化一个 @apidoc[Uri] 类时，查询字符串以其原始字符串形式存储。然后，当您调用`query()`方法时，将从原始字符串中解析出查询字符串。

以下代码说明了如何解析有效的查询字符串。特别是，您可以检查如何使用百分号编码，以及如何解析特殊字符，比如`+`和`;`。

@@@ note
`mode`参数，用于`Query()`和`Uri.query()`，在 @ref[严格和宽松模式](#严格和宽松模式) 中讨论。
@@@

Scala
:   @@snip [UriSpec.scala]($akka-http$/akka-http-core/src/test/scala/akka/http/scaladsl/model/UriSpec.scala) { #query-strict-definition }

Java
:   @@snip [UriTest.java]($akka-http$/akka-http-core/src/test/java/akka/http/javadsl/model/UriTest.java) { #query-strict-definition }


Scala
:   @@snip [UriSpec.scala]($akka-http$/akka-http-core/src/test/scala/akka/http/scaladsl/model/UriSpec.scala) { #query-strict-mode }

Java
:   @@snip [UriTest.java]($akka-http$/akka-http-core/src/test/java/akka/http/javadsl/model/UriTest.java) { #query-strict-mode }

注意：

```
  Uri("http://localhost?a=b").query()
```

等效于：

```
  Query("a=b")
```

正如[章节3.4 of RFC 3986](https://tools.ietf.org/html/rfc3986#section-3.4)，一些特殊的字符，如"/"和"?"被允许在查询字符串里面，不使用("%")的符号来避免他们。

> 斜杠("/")和问号("?")可以表示查询组件中的数据。

当您的URI的查询参数具有另一个URI时，通常使用"/"和"?"。

Scala
:   @@snip [UriSpec.scala]($akka-http$/akka-http-core/src/test/scala/akka/http/scaladsl/model/UriSpec.scala) { #query-strict-without-percent-encode }

Java
:   @@snip [UriTest.java]($akka-http$/akka-http-core/src/test/java/akka/http/javadsl/model/UriTest.java) { #query-strict-without-percent-encode }

但是，某些其他特殊字符可能导致`IllegalUriException`，如果不按如下所示进行百分好编码。

Scala
:   @@snip [UriSpec.scala]($akka-http$/akka-http-core/src/test/scala/akka/http/scaladsl/model/UriSpec.scala) { #query-strict-mode-exception-1 }

Java
:   @@snip [UriTest.java]($akka-http$/akka-http-core/src/test/java/akka/http/javadsl/model/UriTest.java) { #query-strict-mode-exception-1 }


Scala
:   @@snip [UriSpec.scala]($akka-http$/akka-http-core/src/test/scala/akka/http/scaladsl/model/UriSpec.scala) { #query-strict-mode-exception-2 }

Java
:   @@snip [UriTest.java]($akka-http$/akka-http-core/src/test/java/akka/http/javadsl/model/UriTest.java) { #query-strict-mode-exception-2 }

#### 严格和宽松模式

`Uri.query()`方法和`Query()`采用一个参数`mode`，它是`Uri.ParsingMode.Strict`或`Uri.ParsingMode.Relaxed`。切换`mode`会给解析URI中的一些特殊字符带来不同的行为。

Scala
:   @@snip [UriSpec.scala]($akka-http$/akka-http-core/src/test/scala/akka/http/scaladsl/model/UriSpec.scala) { #query-relaxed-mode }

Java
:   @@snip [UriTest.java]($akka-http$/akka-http-core/src/test/java/akka/http/javadsl/model/UriTest.java) { #query-relaxed-definition }

以下两种情况，当你指定`Strict`模式时会抛出`IllegalUriException`：

Scala
:   @@snip [UriSpec.scala]($akka-http$/akka-http-core/src/test/scala/akka/http/scaladsl/model/UriSpec.scala) { #query-strict-mode-exception-1 }

Java
:   @@snip [UriTest.java]($akka-http$/akka-http-core/src/test/java/akka/http/javadsl/model/UriTest.java) { #query-strict-mode-exception-1 #query-strict-mode-exception-2 }

但是`Relaxed`模式会按原样解析它们。

Scala
:   @@snip [UriSpec.scala]($akka-http$/akka-http-core/src/test/scala/akka/http/scaladsl/model/UriSpec.scala) { #query-relaxed-mode-success }

Java
:   @@snip [UriTest.java]($akka-http$/akka-http-core/src/test/java/akka/http/javadsl/model/UriTest.java) { #query-relaxed-mode-success }

但是，即使使用`Relaxed`模式，仍然有无效的特殊字符需要百分比编码。

Scala
:   @@snip [UriSpec.scala]($akka-http$/akka-http-core/src/test/scala/akka/http/scaladsl/model/UriSpec.scala) { #query-relaxed-mode-exception }

Java
:   @@snip [UriTest.java]($akka-http$/akka-http-core/src/test/java/akka/http/javadsl/model/UriTest.java) { #query-relaxed-mode-exception-1 }

除了在参数中指定`mode`之外(例如使用指令时)，您可以在配置中指定`mode`，如下所示。

```
    # Sets the strictness mode for parsing request target URIs.
    # The following values are defined:
    #
    # `strict`: RFC3986-compliant URIs are required,
    #     a 400 response is triggered on violations
    #
    # `relaxed`: all visible 7-Bit ASCII chars are allowed
    #
    uri-parsing-mode = strict
```

要访问URI的查询部分的原始，未解析过的表示，请使用 @apidoc[Uri] 类的`rawQueryString`成员。

#### 提取查询参数的指令

如果你要使用指令提取查询参数，请参阅以下页面。

* @ref:[parameters](../routing-dsl/directives/parameter-directives/parameters.md)
* @ref:[parameter](../routing-dsl/directives/parameter-directives/parameter.md)
