# XML支持

Akka HTTP的 @ref[编组](marshalling.md) 和 @ref[解组](unmarshalling.md) 基础设施使得它比较容易无缝地支持您的数据对象的特定线(wire)表示，如JSON，XML，甚至二进制编码。

@@@ div { .group-java }

Akka HTTP does not currently provide a Java API for XML support. If you need to
produce and consume XML, you can write a @ref[custom marshaller](marshalling.md#自定义编组器)
using [Jackson], which is also the library used for providing @ref[JSON support](json-support.md#jackson-support).

@@ snip [#jackson-xml-support] ($root$/src/test/java/docs/http/javadsl/JacksonXmlSupport.java) { #jackson-xml-support }

The custom XML (un)marshalling code shown above requires that you depend on the `jackson-dataformat-xml` library.

@@dependency [sbt,Gradle,Maven] {
  group="com.fasterxml.jackson.dataformat"
  artifact="jackson-dataformat-xml"
  version="$jackson.version$"
}

@@@

@@@ div { .group-scala }

对于XML，Akka HTTP目前通过其`akka-http-xml`模块直接提供了对[Scala XML][scala-xml]的支持。

## Scala XML 支持

该 @scaladoc[ScalaXmlSupport](akka.http.scaladsl.marshallers.xml.ScalaXmlSupport) 特质提供了一个`FromEntityUnmarshaller[NodeSeq]`和`ToEntityMarshaller[NodeSeq]`，你可以直接使用或者建立。

为了启用对使用[Scala XML][scala-xml] `NodeSeq`进行编组(解组)从和到XML的支持，你必须添加以下依赖项：

@@dependency [sbt,Gradle,Maven] {
  group="com.typesafe.akka"
  artifact="akka-http-xml_$scala.binary.version$"
  version="$project.version$"
}

一旦你完成了XML和类型`NodeSeq`实例之间的编组(解组)，`NodeSeq`应该可以正常且透明地进行工作，通过使用 `import akka.http.scaladsl.marshallers.xml.ScalaXmlSupport._` 或混入 `akka.http.scaladsl.marshallers.xml.ScalaXmlSupport` 特质。

@@@

 [scala-xml]: https://github.com/scala/scala-xml
 [jackson]: https://github.com/FasterXML/jackson
