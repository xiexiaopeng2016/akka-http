# 1. 介绍

Akka HTTP模块在*akka-actor*和*akka-stream*之上实现了完整的服务器端和客户端HTTP堆栈。它不是Web框架，而是提供和使用基于HTTP的服务的更通用的工具包。虽然与浏览器的交互也在范围内，但这并不是Akka HTTP的主要重点。

Akka HTTP遵循一个相当开放的设计，并且很多时候为"做同一件事"提供了几种不同的API级别。您可以选择最适合您的应用程序的API抽象级别。这意味着，如果您在使用高级API实现某件事时遇到问题，则很有可能可以通过低级API完成此任务，这提供了更大的灵活性，但可能需要您编写更多的应用程序代码。

## 哲学

Akka HTTP有明确的目标，其重点是提供用于构建集成层而不是应用程序核心的工具。因此，它认为自己是一套库，而不是一个框架。

就像我们想的那样，一个框架为您提供了一个"框架"，您可以在其中构建你的应用程序。它附带了许多预先制定的决策，并提供了包括支持结构在内的基础，可让您快速入门并快速交付结果。在某种程度上，框架就像骨架一样，您可以在其中放置应用程序的"血肉"，以便让它活起来。因此，如果您在开始应用程序开发之前就选择这些框架，并且在开发过程中尽量坚持框架的"做事方式，那么这些框架的工作效果最好。

例如，如果您要构建一个面向浏览器的Web应用程序，那么选择一个Web框架并在其之上构建应用程序是很有意义的，因为该应用程序的“核心”是浏览器与您在Web上的代码之间的交互作用，服务器。框架制造商选择了一种设计此类应用程序的“行之有效”的方法，并让您“填充”了或多或少灵活的“应用程序模板”的空白。能够依靠这样的最佳实践架构是快速完成工作的重要资产。

For example, if you are building a browser-facing web application it makes sense to choose a web framework and build your application on top of it because the “core” of the application is the interaction of a browser with your code on the web-server. The framework makers have chosen one “proven” way of designing such applications and let you “fill in the blanks” of a more or less flexible “application-template”. Being able to rely on best-practice architecture like this can be a great asset for getting things done quickly.

However, if your application is not primarily a web application because its core is not browser-interaction but some specialized maybe complex business service and you are merely trying to connect it to the world via a REST/HTTP interface a web-framework might not be what you need. In this case the application architecture should be dictated by what makes sense for the core not the interface layer. Also, you probably won’t benefit from the possibly existing browser-specific framework components like view templating, asset management, JavaScript- and CSS generation/manipulation/minification, localization support, AJAX support, etc.

Akka HTTP was designed specifically as “not-a-framework”, not because we don’t like frameworks, but for use cases where a framework is not the right choice. Akka HTTP is made for building integration layers based on HTTP and as such tries to “stay on the sidelines”. Therefore you normally don’t build your application “on top of” Akka HTTP, but you build your application on top of whatever makes sense and use Akka HTTP merely for the HTTP integration needs.

On the other hand, if you prefer to build your applications with the guidance of a framework, you should give [Play Framework](https://www.playframework.com/) or [Lagom](https://www.lagomframework.com/) a try, which both use Akka internally.

## Using Akka HTTP

Akka HTTP is provided as independent modules from Akka itself under its own release cycle. Akka HTTP is @ref[compatible](compatibility-guidelines.md)
with Akka 2.5 and any later 2.x versions released during the lifetime of Akka HTTP 10.1.x. The modules, however, do *not* depend on `akka-actor` or `akka-stream`, so the user is required to
choose an Akka version to run against and add a manual dependency to `akka-stream` of the chosen version.

sbt
:   @@@vars
    ```
    "com.typesafe.akka" %% "akka-http"   % "$project.version$" $crossString$
    "com.typesafe.akka" %% "akka-stream" % "$akka.version$" // or whatever the latest version is
    ```
    @@@

Gradle
:   @@@vars
    ```
    compile group: 'com.typesafe.akka', name: 'akka-http_$scala.binary_version$',   version: '$project.version$'
    compile group: 'com.typesafe.akka', name: 'akka-stream_$scala.binary_version$', version: '$akka.version$'
    ```
    @@@

Maven
:   @@@vars
    ```
    <dependency>
      <groupId>com.typesafe.akka</groupId>
      <artifactId>akka-http_$scala.binary_version$</artifactId>
      <version>$project.version$</version>
    </dependency>
    <dependency>
      <groupId>com.typesafe.akka</groupId>
      <artifactId>akka-stream_$scala.binary_version$</artifactId>
      <version>$akka.version$</version> <!-- Or whatever the latest version is -->
    </dependency>
    ```
    @@@


Alternatively, you can bootstrap a new sbt project with Akka HTTP already
configured using the [Giter8](http://www.foundweekends.org/giter8/) template:

@@@ div { .group-scala }
```sh
sbt -Dsbt.version=0.13.15 new https://github.com/akka/akka-http-scala-seed.g8
```
@@@
@@@ div { .group-java }
```sh
sbt -Dsbt.version=0.13.15 new https://github.com/akka/akka-http-java-seed.g8
```
From there on the prepared project can be built using Gradle or Maven.
@@@

More instructions can be found on the @scala[[template
project](https://github.com/akka/akka-http-scala-seed.g8)]@java[[template
project](https://github.com/akka/akka-http-java-seed.g8)].

## Routing DSL for HTTP servers

The high-level, routing API of Akka HTTP provides a DSL to describe HTTP "routes" and how they should be handled.
Each route is composed of one or more level of @apidoc[Directives] that narrows down to handling one specific type of
request.

For example one route might start with matching the `path` of the request, only matching if it is "/hello", then
narrowing it down to only handle HTTP `get` requests and then `complete` those with a string literal, which
will be sent back as a HTTP OK with the string as response body.

The
@scala[@scaladoc[Route](akka.http.scaladsl.server.index#Route=akka.http.scaladsl.server.RequestContext=%3Escala.concurrent.Future[akka.http.scaladsl.server.RouteResult])]
@java[@javadoc[Route](akka.http.scaladsl.server.Route)]
created using the Route DSL is then "bound" to a port to start serving HTTP requests:

Scala
:   @@snip [HttpServerExampleSpec.scala]($test$/scala/docs/http/scaladsl/HttpServerExampleSpec.scala) { #minimal-routing-example }

Java
:   @@snip [HttpServerMinimalExampleTest.java]($test$/java/docs/http/javadsl/HttpServerMinimalExampleTest.java) { #minimal-routing-example }

When you run this server, you can either open the page in a browser,
at the following url: [http://localhost:8080/hello](http://localhost:8080/hello), or call it in your terminal, via `curl http://localhost:8080/hello`.

## Marshalling

Transforming request and response bodies between over-the-wire formats and objects to be used in your application is
done separately from the route declarations, in marshallers, which are pulled in implicitly using the "magnet" pattern.
This means that you can `complete` a request with any kind of object as long as there is an implicit marshaller
available in scope.

@@@ div { .group-scala }
Default marshallers are provided for simple objects like String or ByteString, and you can define your own for example
for JSON. An additional module provides JSON serialization using the spray-json library (see @ref[JSON Support](common/json-support.md)
for details):

@@dependency [sbt,Gradle,Maven] {
  group="com.typesafe.akka"
  artifact="akka-http-spray-json_$scala.binary.version$"
  version="$project.version$"
}

@@@
@@@ div { .group-java }
JSON support is possible in `akka-http` by the use of Jackson, an external artifact (see @ref[JSON Support](common/json-support.md#jackson-support)
for details):

@@dependency [sbt,Gradle,Maven] {
  group="com.typesafe.akka"
  artifact="akka-http-jackson_$scala.binary.version$"
  version="$project.version$"
}

@@@

A common use case is to reply to a request using a model object having the marshaller transform it into JSON. In
this case shown by two separate routes. The first route queries an asynchronous database and marshalls the
@scala[`Future[Option[Item]]`]@java[`CompletionStage<Optional<Item>>`] result into a JSON response. The second unmarshalls an `Order` from the incoming request,
saves it to the database and replies with an OK when done.

Scala
:   @@snip [SprayJsonExampleSpec.scala]($test$/scala/docs/http/scaladsl/SprayJsonExampleSpec.scala) { #second-spray-json-example }

Java
:   @@snip [JacksonExampleTest.java]($test$/java/docs/http/javadsl/JacksonExampleTest.java) { #second-jackson-example }

When you run this server, you can update the inventory via `curl -H "Content-Type: application/json" -X POST -d '{"items":[{"name":"hhgtg","id":42}]}' http://localhost:8080/create-order` on your terminal - adding an item named `"hhgtg"` and having an `id=42`; and then view the inventory either in a browser, at a url like: [http://localhost:8080/item/42](http://localhost:8080/item/42) - or on the terminal,
via `curl http://localhost:8080/item/42`.

The logic for the marshalling and unmarshalling JSON in this example is provided by the @scala["spray-json"]@java["Jackson"] library
(details on how to use that here: @scala[@ref[JSON Support](common/json-support.md))]@java[@ref[JSON Support](common/json-support.md#jackson-support))].

## Streaming

One of the strengths of Akka HTTP is that streaming data is at its heart meaning that both request and response bodies
can be streamed through the server achieving constant memory usage even for very large requests or responses. Streaming
responses will be backpressured by the remote client so that the server will not push data faster than the client can
handle, streaming requests means that the server decides how fast the remote client can push the data of the request
body.

Example that streams random numbers as long as the client accepts them:

Scala
:   @@snip [HttpServerExampleSpec.scala]($test$/scala/docs/http/scaladsl/HttpServerExampleSpec.scala) { #stream-random-numbers }

Java
:   @@snip [HttpServerStreamRandomNumbersTest.java]($test$/java/docs/http/javadsl/HttpServerStreamRandomNumbersTest.java) { #stream-random-numbers }

Connecting to this service with a slow HTTP client would backpressure so that the next random number is produced on
demand with constant memory usage on the server. This can be seen using curl and limiting the rate
`curl --limit-rate 50b 127.0.0.1:8080/random`

Akka HTTP routes easily interacts with actors. In this example one route allows for placing bids in a fire-and-forget
style while the second route contains a request-response interaction with an actor. The resulting response is rendered
as json and returned when the response arrives from the actor.

Scala
:   @@snip [HttpServerExampleSpec.scala]($test$/scala/docs/http/scaladsl/HttpServerExampleSpec.scala) { #actor-interaction }

Java
:   @@snip [HttpServerActorInteractionExample.java]($test$/java/docs/http/javadsl/HttpServerActorInteractionExample.java) { #actor-interaction }

When you run this server, you can add an auction bid via `curl -X PUT http://localhost:8080/auction?bid=22&user=MartinO` on the terminal; and then you can view the auction status either in a browser, at the url [http://localhost:8080/auction](http://localhost:8080/auction), or, on the terminal, via `curl http://localhost:8080/auction`.

More details on how JSON marshalling and unmarshalling works can be found in the @ref[JSON Support section](common/json-support.md).

Read more about the details of the high level APIs in the section @ref[High-level Server-Side API](routing-dsl/index.md).

## Low-level HTTP server APIs

The low-level Akka HTTP server APIs allows for handling connections or individual requests by accepting
@apidoc[HttpRequest] s and answering them by producing @apidoc[HttpResponse] s. This is provided by the `akka-http-core` module,
which is included automatically when you depend on `akka-http` but can also be used on its own.
APIs for handling such request-responses as function calls and as a @apidoc[Flow[HttpRequest, HttpResponse, \_]] are available.

Scala
:   @@snip [HttpServerExampleSpec.scala]($test$/scala/docs/http/scaladsl/HttpServerExampleSpec.scala) { #low-level-server-example }

Java
:   @@snip [HttpServerLowLevelExample.java]($test$/java/docs/http/javadsl/HttpServerLowLevelExample.java) { #low-level-server-example }

Read more details about the low level APIs in the section @ref[Core Server API](server-side/low-level-api.md).

## HTTP client API

The client APIs provide methods for calling a HTTP server using the same @apidoc[HttpRequest] and @apidoc[HttpResponse] abstractions
that Akka HTTP server uses but adds the concept of connection pools to allow multiple requests to the same server to be
handled more performantly by re-using TCP connections to the server.

Example simple request:

Scala
:   @@snip [HttpClientExampleSpec.scala]($test$/scala/docs/http/scaladsl/HttpClientExampleSpec.scala) { #single-request-example }

Java
:   @@snip [ClientSingleRequestExample.java]($test$/java/docs/http/javadsl/ClientSingleRequestExample.java) { #single-request-example }

Read more about the details of the client APIs in the section @ref[Consuming HTTP-based Services (Client-Side)](client-side/index.md).

## The modules that make up Akka HTTP

Akka HTTP is structured into several modules:

akka-http
: Higher-level functionality, like (un)marshalling, (de)compression as well as a powerful DSL
for defining HTTP-based APIs on the server-side, this is the recommended way to write HTTP servers
with Akka HTTP. Details can be found in the section @ref[High-level Server-Side API](routing-dsl/index.md)

akka-http-core
: A complete, mostly low-level, server- and client-side implementation of HTTP (incl. WebSockets)
Details can be found in sections @ref[Core Server API](server-side/low-level-api.md) and @ref[Consuming HTTP-based Services (Client-Side)](client-side/index.md)

akka-http-testkit
: A test harness and set of utilities for verifying server-side service implementations


@@@ div { .group-scala }
akka-http-spray-json
: Predefined glue-code for (de)serializing custom types from/to JSON with [spray-json](https://github.com/spray/spray-json)
Details can be found here: @ref[JSON Support](common/json-support.md)
@@@

@@@ div { .group-scala }
akka-http-xml
: Predefined glue-code for (de)serializing custom types from/to XML with [scala-xml](https://github.com/scala/scala-xml)
Details can be found here: @ref[XML Support](common/xml-support.md)
@@@
@@@ div { .group-java }
akka-http-jackson
: Predefined glue-code for (de)serializing custom types from/to JSON with [jackson](https://github.com/FasterXML/jackson)
@@@
