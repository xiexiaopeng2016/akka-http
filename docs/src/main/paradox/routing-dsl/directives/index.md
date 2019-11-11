# 指令

“指令”是用于创建任意复杂的 @ref[路由结构](../routes.md) 的小构件。Akka HTTP已经预定义了大量指令，您也可以轻松地构建自己的指令:

@@toc { depth=1 }

@@@ index

* [alphabetically](alphabetically.md)
* [by-trait](by-trait.md)
* [custom-directives](custom-directives.md)

@@@

## 基础

指令创建 @ref[路由](../routes.md)。要了解指令的工作方式，将它们与创建路径的"原始"方式进行对比会很有帮助。

有效的 @ref[路由](../routes.md)只是高度专业化的功能，它们接受一个 @apidoc[RequestContext]并最终`complete`它，它可能(通常应该)异步发生。

@@@ div { .group-java }

The @ref[complete](route-directives/complete.md) directive simply completes the request with a response:

@@@

@@@ div { .group-scala }


由于 @scala[@scaladoc[Route](akka.http.scaladsl.server.index#Route=akka.http.scaladsl.server.RequestContext=%3Escala.concurrent.Future[akka.http.scaladsl.server.RouteResult])]只是函数类型的类型别名，Route实例可以以任何方式写入，其中可以写入函数实例，例如作为函数字面量：

```scala
val route: Route = { ctx => ctx.complete("yeah") }
```

或更短:

```scala
val route: Route = _.complete("yeah")
```

使用 @ref[complete](route-directives/complete.md)指令，它变得更短：

@@@

Scala
:  ```scala
val route = complete("yeah")
```

Java
:  ```java
Route route = complete("yeah");
```

@@@ div { .group-scala }

这三种编写此 @scala[@scaladoc[Route](akka.http.scaladsl.server.index#Route=akka.http.scaladsl.server.RequestContext=%3Escala.concurrent.Future[akka.http.scaladsl.server.RouteResult])]@java[@apidoc[Route]]的方式完全等效，在所有情况下创建的`route`行为都相同。

让我们看一个稍微复杂的示例，以特别强调一个重要点。考虑以下两条路由：

```scala
val a: Route = {
  println("MARK")
  ctx => ctx.complete("yeah")
}

val b: Route = { ctx =>
  println("MARK")
  ctx.complete("yeah")
}
```

`a`和`b`之间的区别在于何时执行`println`语句。在`a`的情况下，当路由被构造时它被执行 *一次*，而在`b`情况下，路由 *运行* 时，它每次都被执行。

使用 @ref[complete](route-directives/complete.md)指令，可以达到以下相同的效果：

```scala
val a = {
  println("MARK")
  complete("yeah")
}

val b = complete {
  println("MARK")
  "yeah"
}
```

这是可行的，因为 @ref[complete](route-directives/complete.md)指令的参数是 *按名称* 评估的，即每次运行生成的路由时都会对其进行重新评估。

让我们更进一步：

```scala
val route: Route = { ctx =>
  if (ctx.request.method == HttpMethods.GET)
    ctx.complete("Received GET")
  else
    ctx.complete("Received something else")
}
```

使用 @ref[get](method-directives/get.md)和 @ref[complete](route-directives/complete.md)指令，我们可以这样编写这个路由：

```scala
val route =
  concat(
    get {
      complete("Received GET")
    },
    complete("Received something else")
  )
```

同样，生成的路由在所有情况下都具有相同的行为。

请注意，如果您愿意，也可以混合两种风格的路由创建方式：

```scala
val route =
  concat(
    get { ctx =>
      ctx.complete("Received GET")
    },
    complete("Received something else")
  )
```

在这里, @ref[get](method-directives/get.md)指令的内部路由被编写为显式函数字面量。

然而，正如您从这些示例中看到的，使用指令而不是"手工"构建路由会使代码更加简洁，因此更具可读性和可维护性。另外，它提供了更好的可组合性(正如您将在接下来的部分中看到的)。因此，在使用Akka HTTP的路由DSL时，您几乎永远都不必回过头来直接操作 @ref[RequestContext](../routes.md#requestcontext)通过 @scala[@scaladoc[Route](akka.http.scaladsl.server.index#Route=akka.http.scaladsl.server.RequestContext=%3Escala.concurrent.Future[akka.http.scaladsl.server.RouteResult])]@java[@apidoc[Route]]函数字面量来创建路由。

@@@

@@@ div { .group-java }

Writing multiple routes that are tried as alternatives (in-order of definition), is as simple as using the `concat(route1, route2)`,
method:

```java
Route routes = concat(
  pathSingleSlash(() ->
    getFromResource("web/calculator.html")
  ),
  path("hello", () -> complete("World!))
);
```

You could also simply define a "catch all" completion by providing it as the last route to attempt to match.
In the example below we use the `get()` (one of the @ref[MethodDirectives](method-directives/index.md)) to match all incoming `GET`
requests for that route, and all other requests will be routed towards the other "catch all" route, that completes the route:

```java
Route route =
  get(
    () -> complete("Received GET")
  ).orElse(
    () -> complete("Received something else")
  )
```

@@@

如果没有路由匹配给定的请求，则将返回一个默认`404 Not Found`响应作为响应。

## 结构体

指令的一般结构如下：

Scala
:  ```scala
name(arguments) { extractions =>
  ... // inner route
}
```

Java
:  ```java
directiveName(arguments [, ...], (extractions [, ...]) -> {
  ... // inner route
})
```

它具有名称，零个或多个参数以及可选的内部路由( @ref[RouteDirectives](route-directives/index.md)的特殊之处在于它们始终在leaf级别使用，因此不能具有内部路由)。

另外，指令可以"提取"一些值，并使它们可作为函数参数供其内部路由使用。当"从外部"看到时，带有内部路由的指令形成 @scala[@scaladoc[Route](akka.http.scaladsl.server.index#Route=akka.http.scaladsl.server.RequestContext=%3Escala.concurrent.Future[akka.http.scaladsl.server.RouteResult])]@java[@apidoc[Route]]类型的表达式。

## 指令做什么

指令可以执行以下一项或多项操作：

 * 在传递到其内部路由之前，将传入的 @apidoc[RequestContext]进行转换(即，修改请求)
 * 根据某些逻辑过滤 @apidoc[RequestContext]，即仅传递某些请求而拒绝其他请求
 * 从 @apidoc[RequestContext] 中提取值，并将其作为"extractions"提供给其内部路由
 * 将一些逻辑链接到 @ref[RouteResult](../routes.md#routeresult)，将来的转换链(即修改响应或拒绝)
 * 完成(complete)请求

这意味着一个`Directive`完全包装了其内部路由的功能，并且可以在请求和响应端(或两者)应用任意复杂的转换。

## 组合指令

正如您从目前给出的示例中所看到的，组合指令的"正常"方式是嵌套。让我们来看看这个具体的例子:

Scala
:  @@snip [DirectiveExamplesSpec.scala]($test$/scala/docs/http/scaladsl/server/DirectiveExamplesSpec.scala) { #example-1 }

Java
:  @@snip [DirectiveExamplesTest.java]($test$/java/docs/http/javadsl/server/DirectiveExamplesTest.java) { #example1 }

在这里，`get`和`put`指令 @scala[与`concat`组合器]@java[使用`orElse`方法]链接在一起，形成了更高级别的路由，该路由充当了`path`指令的内部路由。让我们用以下方式重写它：

Scala
:  @@snip [DirectiveExamplesSpec.scala]($test$/scala/docs/http/scaladsl/server/DirectiveExamplesSpec.scala) { #getOrPut }

Java
:  @@snip [DirectiveExamplesTest.java]($test$/java/docs/http/javadsl/server/DirectiveExamplesTest.java) { #getOrPut }

@@@ div { .group-java }

In this previous example, we combined the `get` and `put` directives into one composed directive and extracted it to its own method, which could be reused anywhere else in our code.

Instead of extracting the composed directives to its own method, we can also use the available `anyOf` combinator. The following code is equivalent to the previous one:

@@@

@@@ div { .group-scala }

从此片段中看不到的是，指令不是作为简单方法实现的，而是作为`Directive`类型的独立对象实现的。这为您在编写指令时提供了更大的灵活性。例如，你也可以在指令上使用`|`操作符。这是编写示例的另一种方法：

@@@

Scala
:  @@snip [DirectiveExamplesSpec.scala]($test$/scala/docs/http/scaladsl/server/DirectiveExamplesSpec.scala) { #getOrPutUsingPipe }

Java
:  @@snip [DirectiveExamplesTest.java]($test$/java/docs/http/javadsl/server/DirectiveExamplesTest.java) { #getOrPutUsingAnyOf }

@@@ div { .group-scala }

或更好（无需深入手动编写一个显式 @scala[@scaladoc[Route](akka.http.scaladsl.server.index#Route=akka.http.scaladsl.server.RequestContext=%3Escala.concurrent.Future[akka.http.scaladsl.server.RouteResult])]@java[@apidoc[Route]]函数)：

@@snip [DirectiveExamplesSpec.scala]($test$/scala/docs/http/scaladsl/server/DirectiveExamplesSpec.scala) { #getOrPutUsingPipeAndExtractMethod }

如果您有较大的路由结构，其中`(get | put)`代码段会出现多次，那么您也可以像这样将其分解：

@@snip [DirectiveExamplesSpec.scala]($test$/scala/docs/http/scaladsl/server/DirectiveExamplesSpec.scala) { #example-5 }

请注意，由于`getOrPut`不接受任何参数，因此这里可以是`val`。

作为嵌套的替代方法，您还可以使用`&`运算符：

@@snip [DirectiveExamplesSpec.scala]($test$/scala/docs/http/scaladsl/server/DirectiveExamplesSpec.scala) { #example-6 }

这里您可以看到，当提取产生的指令与`&`组合时，产生的"超级指令"只是提取其子提取的连接。

再一次，如果你想的话，你可以将要素分解出来，从而把指令配置的"分解"推向极致

@@snip [DirectiveExamplesSpec.scala]($test$/scala/docs/http/scaladsl/server/DirectiveExamplesSpec.scala) { #example-7 }

这种类型的组合指令使用`|`和`&`运算符和"保存"更复杂的指令配置为"val"一样，所有指令都采用内部路由。

@@@

@@@ div { .group-java }

The previous example, tries to complete the route first with a `GET` or with a `PUT` if the first one was rejected. 

In case you are constantly nesting the same directives several times in you code, you could factor them out in their own method and use it everywhere:

@@snip [DirectiveExamplesTest.java]($test$/java/docs/http/javadsl/server/DirectiveExamplesTest.java) { #composeNesting }

Here we simple created our own combined directive that accepts `GET` requests, then extracts the method and completes it with an inner route that takes this HTTP method as a parameter.

Again, instead of extracting own combined directives to its own method, we can make use of the `allOf` combinator. The following code is equivalent to the previous one:

@@snip [DirectiveExamplesTest.java]($test$/java/docs/http/javadsl/server/DirectiveExamplesTest.java) { #composeNestingAllOf }

In this previous example, the the inner route function provided to `allOf` will be called when the request is a `GET` and with the extracted client IP obtained from the second directive.

As you have already seen in the previous section, you can also use the `concat` method defined in @apidoc[RouteDirectives] as an alternative to `orElse` chaining. Here you can see the first example again, rewritten using `concat`:

@@snip [DirectiveExamplesTest.java]($test$/java/docs/http/javadsl/server/DirectiveExamplesTest.java) { #usingConcat }

The `concat` combinator comes handy when you want to avoid nesting. Here you can see an illustrative example:
 
@@snip [DirectiveExamplesTest.java]($test$/java/docs/http/javadsl/server/DirectiveExamplesTest.java) { #usingConcatBig }

Notice how you could adjust the indentation in these last two examples to have a more readable code.

@@@

请注意，将多个指令"压缩"到一个指令中太过复杂，可能不会产生可读性最好、更容易维护的路由代码。甚至本系列的第一个示例实际上可能是最易读的。

尽管如此，这里给出的练习的目的是向您展示指令可以有多么灵活，以及您如何使用它们的能力在适合**您的**应用程序的抽象级别上定义您的web服务行为。

@@@ div { .group-scala }

### 使用`~`运算符组合指令

@@@ 

@@@ note { .group-scala }
不可思议：忘记指令之间的`~`(波浪号)字符会导致完全有效的Scala代码能够编译，但无法按预期工作。原本打算作为单个表达式，实际上是多个表达式，而且只有最后一个表达式将作为父指令的结果使用。因此，建议的组合路由方法是使用`concat`组合器。
@@@

@@@ div { .group-scala }

或者，我们可以使用`~`操作符将指令组合在一起，而不是将每个指令作为单独的参数传递。让我们来看看这个组合符的用法：

@@snip [DirectiveExamplesSpec.scala]($test$/scala/docs/http/scaladsl/server/DirectiveExamplesSpec.scala) { #example-8 }

@@@

## 指令的类型安全

当您使用 @scala[`|`和`&`操作符]@java[`anyOf`和`allOf`方法]组合指令时，路由DSL确保所有提取均按预期进行，并且在编译时强制执行逻辑约束。

例如，您不能 @scala[`|`]@java[`anyOf`]一个指令生成一个提取：

Scala
:   ```scala
val route = path("order" / IntNumber) | get // doesn't compile
```

Java
:  ```java
anyOf(this::get, this::extractClientIP, routeProvider) // doesn't compile
```

此外，提取的数量和类型必须匹配:

Scala
:  ```scala
val route = path("order" / IntNumber) | path("order" / DoubleNumber)   // doesn't compile
val route = path("order" / IntNumber) | parameter('order.as[Int])      // ok
```

Java
:  ```java
anyOf(this::extractClientIP, this::extractMethod, routeProvider) // doesn't compile
anyOf(bindParameter(this::parameter, "foo"), bindParameter(this::parameter, "bar"), routeProvider) // ok
```
In this previous example we make use of the `bindParameter` function located in `akka-http/akka.http.javadsl.common.PartialApplication`.
In order to be able to call `anyOf`, we need to convert our directive that takes 2 parameters to a function that takes only 1.
In this particular case we want to use the `parameter` directive that takes a `String` and a function from `String` to @scala[@scaladoc[Route](akka.http.scaladsl.server.index#Route=akka.http.scaladsl.server.RequestContext=%3Escala.concurrent.Future[akka.http.scaladsl.server.RouteResult])]@java[@apidoc[Route]],
so to be able to use it in combination with `anyOf`, we need to bind the first parameter to `foo` and to `bar` in the second one. `bindParameter(this::parameter, "foo")` is equivalent 
to define your own function like this:
```java
Route parameterFoo(Function<String, Route> inner) {
  return parameter("foo", inner);
}
```

当您将产生指令生成的提取与 @scala[`&`运算符]@java[`allOf`方法] 组合在一起时，将正确收集所有提取：

Scala
:  ```scala
val order = path("order" / IntNumber) & parameters('oem, 'expired ?)
val route =
  order { (orderId, oem, expired) =>
    ...
  }
```

Java
:  ```java
allOf(this::extractScheme, this::extractMethod, (scheme, method) -> ...) 
```

指令提供了一种很好的方式，以即插即用的方式从小的构建块构建web服务逻辑，同时保持DRY状态和完全类型安全。

如果各种各样的 @ref[预先定义的指令](alphabetically.md) 不能完全满足您的需求，您还可以轻松创建 @ref[自定义指令](custom-directives.md)。

@@@ div { .group-scala }

## 自动元组提取(扁平化)

在 @ref[基础](#基础) 和 @ref[组合指令](#组合指令) 中描述的便捷Scala DSL语法使内部的元组提取成为可能. 让我们通过例子来看看它是如何工作的。
 
```scala
val futureOfInt: Future[Int] = Future.successful(1)
val route =
  path("success") {
    onSuccess(futureOfInt) { //: Directive[Tuple1[Int]]
      i => complete("Future was completed.")
    }
  }
```
看看上面的代码，`onSuccess(futureOfInt)` 返回一个 `Directive1[Int] = Directive[Tuple1[Int]]`。

```scala
val futureOfTuple2: Future[Tuple2[Int,Int]] = Future.successful( (1,2) )
val route =
  path("success") {
    onSuccess(futureOfTuple2) { //: Directive[Tuple2[Int,Int]]
      (i, j) => complete("Future was completed.")
    }
  }
```

类似地，`onSuccess(futureOfTuple2)` 返回一个`Directive1[Tuple2[Int,Int]] = Directive[Tuple1[Tuple2[Int,Int]]]`，
但是这个会自动转换成`Directive[Tuple2[Int,Int]]`以避免嵌套元组。

```scala
val futureOfUnit: Future[Unit] = Future.successful( () )
val route =
  path("success") {
    onSuccess(futureOfUnit) { //: Directive0
        complete("Future was completed.")
    }
  }
```

如果将来返回 `Future[Unit]`， 这是一个有点特殊的情况，当它的结果是`Directive0`时。看看上面的代码, `onSuccess(futureOfUnit)` 返回一个 `Directive1[Unit] = Directive[Tuple1[Unit]]`。但是，DSL将 `Unit` 翻译为 `Tuple0`，并自动将结果转化为 `Directive[Unit] = Directive0`。

@@@
