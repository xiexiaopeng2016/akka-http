# 路径匹配器DSL

为了能够有效地使用 @ref[PathDirectives](directives/path-directives/index.md)，您应该对`PathMatcher`小型DSL有一些了解，那里Akka HTTP为优雅地定义URI匹配行为提供了支持。

## 概述

当一个请求(或相应的 @apidoc[RequestContext] 实例)进入路由结构时，它具有与`request.uri.path`完全相同的"未匹配路径"" 。当它下降到路由树并通过一个或多个 @ref[pathPrefix](directives/path-directives/pathPrefix.md) 或 @ref[path](directives/path-directives/path.md) 指令传递时，"未匹配的路径"将从左侧逐渐被"吞噬"，直到在大多数情况下最终被完全消耗为止。

那些不但完全匹配并消耗而且从每个指令中未匹配的路径中提取的东西，是通过路径匹配DSL定义的，它们是围绕这些类型构建的：

Scala
:   ```scala
    trait PathMatcher[L: Tuple]
    type PathMatcher0 = PathMatcher[Unit]
    type PathMatcher1[T] = PathMatcher[Tuple1[T]]
    type PathMatcher2[T,U] = PathMatcher[Tuple2[T,U]]
    // .. etc
    ```

Java
:   ```java
    package akka.http.javadsl.server;
    class PathMatcher0
    class PathMatcher1<T1>
    class PathMatcher2<T1, T2>
    // .. etc
    ```

`PathMatcher`实例提取的值的数量和类型 @scala[由`L`类型参数表示，它必须是Scala的TupleN类型之一或`Unit`(由`Tuple`上下文绑定指定)。]@java[is determined by the class and its type parameters.] @scala[便捷别名`PathMatcher0`可以用于所有不提取任何内容的匹配器，同时`PathMatcher1[T]`定义仅提取单个`T`类型值的匹配器。]

这是一个更复杂的`PathMatcher`表达式的示例：

Scala
:  @@snip [PathDirectivesExamplesSpec.scala]($test$/scala/docs/http/scaladsl/server/directives/PathDirectivesExamplesSpec.scala) { #path-matcher }

Java
:  @@snip [PathDirectivesExamplesTest.java]($test$/java/docs/http/javadsl/server/directives/PathDirectivesExamplesTest.java) { #path-matcher }

这将匹配诸如`foo/bar/X42/edit`或 @scala[`foo/bar/X/create`]@java[`foo/bar/X37/create`] 的路径。

@@@ note
路径匹配DSL描述了URL解码 **之后** 要接受的路径。这就是为什么路径分隔斜线具有特殊的状态，不能简单地指定为字符串的一部分。字符串"foo/bar"将与原始URI路径"foo%2Fbar"匹配，这很可能不是您想要的！
@@@



## 基本路径匹配器

@@@ div { .group-scala }
一个复杂的`PathMatcher`可以通过组合或修改更基本的来构建。以下是Akka HTTP已为您提供的基本匹配器：

String
: 您可以将一个`String`实例用作一个`PathMatcher0`。字符串只是简单地匹配自己而不会提取任何值。注意，字符串被解释为路径的解码表示形式，因此，如果它们包含'/'字符，则此字符将与编码后的原始URI中的"%2F"匹配！

Regex
: 您可以将一个`Regex`实例用作一个`PathMatcher1[String]`，其与任何正则表达式匹配并提取一个`String`值。一个`PathMatcher`创建自一个正则表达式，提取任一完全匹配(如果正则表达式不包含捕获组)或捕获组(如果该正则表达式正好包含一个捕获组)。如果正则表达式包含多个捕获组，将抛出`IllegalArgumentException`。

Map[String, T]
: 您可以将一个`Map[String, T]`实例用作`PathMatcher1[T]`，与任何键匹配并为其提取相应的映射值。

Slash: PathMatcher0
: 恰好匹配一个路径分隔斜杠(`/`)字符，并且不提取任何内容。

Segment: PathMatcher1[String]
: 如果未匹配的路径以路径段(即非斜线)开头，则匹配。如果是这样，则将路径段(segment)提取为一个`String`实例。

PathEnd: PathMatcher0
: 匹配路径的最后，类似于`$`正则表达式，不提取任何内容。

Remaining: PathMatcher1[String]
: 匹配并提取请求的URI路径中完整剩余的未匹配部分，作为一个(已编码!)字符串。如果您需要访问路径的剩余 *解码* 元素，请改用`RemainingPath`。

RemainingPath: PathMatcher1[Path]
: 匹配并提取完整剩余，请求URI路径中未匹配的部分。

IntNumber: PathMatcher1[Int]
: 有效地匹配一个十进制(无符号)数字，并提取其(非负)`Int`值。匹配器将不匹配数字0或一系列表示的`Int`值大于`Int.MaxValue`的数字。

LongNumber: PathMatcher1[Long]
: Efficiently matches a number of decimal digits (unsigned) and extracts their (non-negative) `Long` value. The matcher
will not match zero digits or a sequence of digits that would represent an `Long` value larger than `Long.MaxValue`.

HexIntNumber: PathMatcher1[Int]
: Efficiently matches a number of hex digits and extracts their (non-negative) `Int` value. The matcher will not match
zero digits or a sequence of digits that would represent an `Int` value larger than `Int.MaxValue`.

HexLongNumber: PathMatcher1[Long]
: Efficiently matches a number of hex digits and extracts their (non-negative) `Long` value. The matcher will not
match zero digits or a sequence of digits that would represent an `Long` value larger than `Long.MaxValue`.

DoubleNumber: PathMatcher1[Double]
: 匹配并提取一个`Double`值。匹配的字符串表示形式是纯小数，可以是带双精度值的可选带符号形式，即没有指数。

JavaUUID: PathMatcher1[UUID]
: 匹配并提取一个`java.util.UUID`实例。

Neutral: PathMatcher0
: 始终匹配的匹配器，不消耗任何东西，也不提取任何东西。主要用作`PathMatcher`组合中的中立元素。

Segments: PathMatcher1[List[String]]
: 将所有剩余的段匹配为字符串列表。请注意，这也可以是"无段"，从而导致列表为空。如果路径中带有尾部斜杠，则该斜杠将 *不* 匹配，即保持未匹配，并由潜在的嵌套指令消耗。

separateOnSlashes(string: String): PathMatcher0
: 将包含斜杠的路径字符串转换为一个`PathMatcher0`，其将斜杠解释为路径段分隔符。这意味着无法使用此帮助程序构造与"%2F"匹配的匹配器。

provide[L: Tuple](extractions: L): PathMatcher[L]
: 始终匹配，不消耗任何东西并提取给定`TupleX`的值。

PathMatcher[L: Tuple](prefix: Path, extractions: L): PathMatcher[L]
: 匹配并消耗给定的路径前缀，并提取给定的提取列表。如果给定的前缀为空，则返回的匹配器将始终匹配且不消耗任何内容。

@@@

@@@ div { .group-java }

A path matcher is a description of a part of a path to match. The simplest path matcher is `PathMatcher.segment` which
matches exactly one path segment against the supplied constant string.

Other path matchers defined in @apidoc[PathMatchers] match the end of the path (`PathMatchers.END`), a single slash
(`PathMatchers.SLASH`), or nothing at all (`PathMatchers.NEUTRAL`).

Many path matchers are hybrids that can both match (by using them with one of the PathDirectives) and extract values,
Extracting a path matcher value (i.e. using it with `handleWithX`) is only allowed if it nested inside a path
directive that uses that path matcher and so specifies at which position the value should be extracted from the path.

Predefined path matchers allow extraction of various types of values:

PathMatchers.segment(String)
: Strings simply match themselves and extract no value.
Note that strings are interpreted as the decoded representation of the path, so if they include a '/' character
this character will match "%2F" in the encoded raw URI!

PathMatchers.regex
: You can use a regular expression instance as a path matcher, which matches whatever the regex matches and extracts
one `String` value. A `PathMatcher` created from a regular expression extracts either the complete match (if the
regex doesn't contain a capture group) or the capture group (if the regex contains exactly one capture group).
If the regex contains more than one capture group an `IllegalArgumentException` will be thrown.

PathMatchers.SLASH
: Matches exactly one path-separating slash (`/`) character.

PathMatchers.END
: Matches the very end of the path, similar to `$` in regular expressions.

PathMatchers.Segment
: Matches if the unmatched path starts with a path segment (i.e. not a slash).
If so the path segment is extracted as a `String` instance.

PathMatchers.Remaining
: Matches and extracts the complete remaining unmatched part of the request's URI path as an (encoded!) String.
If you need access to the remaining *decoded* elements of the path use `RemainingPath` instead.

PathMatchers.intValue
: Efficiently matches a number of decimal digits (unsigned) and extracts their (non-negative) `Int` value. The matcher
will not match zero digits or a sequence of digits that would represent an `Int` value larger than `Integer.MAX_VALUE`.

PathMatchers.longValue
: Efficiently matches a number of decimal digits (unsigned) and extracts their (non-negative) `Long` value. The matcher
will not match zero digits or a sequence of digits that would represent an `Long` value larger than `Long.MAX_VALUE`.

PathMatchers.hexIntValue
: Efficiently matches a number of hex digits and extracts their (non-negative) `Int` value. The matcher will not match
zero digits or a sequence of digits that would represent an `Int` value larger than `Integer.MAX_VALUE`.

PathMatchers.hexLongValue
: Efficiently matches a number of hex digits and extracts their (non-negative) `Long` value. The matcher will not
match zero digits or a sequence of digits that would represent an `Long` value larger than `Long.MAX_VALUE`.

PathMatchers.uuid
: Matches and extracts a `java.util.UUID` instance.

PathMatchers.NEUTRAL
: A matcher that always matches, doesn't consume anything and extracts nothing.
Serves mainly as a neutral element in `PathMatcher` composition.

PathMatchers.segments
: Matches all remaining segments as a list of strings. Note that this can also be "no segments" resulting in the empty
list. If the path has a trailing slash this slash will *not* be matched, i.e. remain unmatched and to be consumed by
potentially nested directives.

@@@

@@@ div { .group-scala }
## 组合器

路径匹配器可以与以下组合器组合在一起以形成更高级别的结构：

波浪运算符(`~`)
: 波浪号是最基本的组合器。它只是将两个匹配器简单的连接成一个，即如果第一个匹配(并被消耗了)，则尝试第二个。两个匹配器的提取会安全地进行类型组合。例如：`"foo" ~ "bar"`产生一个与`"foobar"`相同的匹配器。

斜杠运算符(`/`)
: 该运算符连接两个匹配器，并在它们之间插入一个`Slash`匹配器。例如：`"foo" / "bar"`与`"foo" ~ Slash ~ "bar"`相同。

管道运算符(`|`)
: 这个操作符组合了两个匹配器选项，因为只有当第一个匹配器不匹配时，才会尝试第二个匹配器。

此运算符组合了两个匹配器替代项，因为只有在第一个 *不* 匹配时才尝试第二个。两个子匹配器必须具有兼容的类型。例如：`"foo" | "bar"`将匹配"foo" *或* "bar"。将使用此运算符表示的替代项与一个`/`运算符组合时，请确保用括号将替代项括起来，如下所示：`("foo" | "bar") / "bom"`。否则，`/`运算符将具有优先权，并且仅适用于替代项的右侧。

## 修饰符

路径匹配器实例可以使用以下修饰符方法进行转换：
Path matcher instances can be transformed with these modifier methods:

斜杠(`/`)
: 斜杠运算符不仅可以用作组合两个匹配器实例的组合器，还可以用作后缀调用。`matcher /`等同于`matcher ~ Slash`但更短且更易于阅读。

?
: 通过将一个匹配器后缀`?`，您可以将任何`PathMatcher`转换为始终匹配的一个，可选地消耗和可能提取一个放在下面的匹配器提取的`Option`。结果类型取决于放在下面的匹配器的类型：

|If a `matcher` is of type | then `matcher.?` is of type|
|--------------------------|----------------------------|
|`PathMatcher0`          | `PathMatcher0`           |
|`PathMatcher1[T]`       | `PathMatcher1[Option[T]]`|
|`PathMatcher[L: Tuple]` | `PathMatcher[Option[L]]` |

repeat(separator: PathMatcher0 = PathMatchers.Neutral)
: 通过将一个匹配器后缀`repeat(separator)`，您可以将任何`PathMatcher`变成始终匹配的一个，消耗零次或多次(使用给定的分隔符)，并有可能提取一个放在下面的匹配器提取的`List`。结果类型取决于放在下面的匹配器的类型：

|If a `matcher` is of type | then `matcher.repeat(...)` is of type|
|--------------------------|--------------------------------------|
|`PathMatcher0`          | `PathMatcher0`         |
|`PathMatcher1[T]`       | `PathMatcher1[List[T]]`|
|`PathMatcher[L: Tuple]` | `PathMatcher[List[L]]` |

unary_!
: 通过给一个匹配器加前缀`!`，它可以变成一个`PathMatcher0`，只有当放在下面的匹配器 *不* 匹配时才匹配，反之亦然。

transform / (h)flatMap / (h)map
: These modifiers allow you to append your own "post-application" logic to another matcher in order to form a custom
one. You can map over the extraction(s), turn mismatches into matches or vice-versa or do anything else with the
results of the underlying matcher. Take a look at the method signatures and implementations for more guidance as to
how to use them.

@@@

## 示例

下面是一些路径匹配的例子:

Scala
:   @@snip [PathDirectivesExamplesSpec.scala]($test$/scala/docs/http/scaladsl/server/directives/PathDirectivesExamplesSpec.scala) { #path-dsl }

Java
:   @@snip [PathDirectiveExampleTest.java]($test$/java/docs/http/javadsl/server/PathDirectiveExampleTest.java) { #path-examples }

