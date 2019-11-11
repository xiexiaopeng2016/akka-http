# 基础指令

基本指令是构建 @ref[自定义指令](../custom-directives.md) 的构建块。因此，它们通常不直接用于路由，而是用于定义新指令。

<a id="providedirectives"></a>
## 为内部路由提供值

这些指令通过提取为内部路由提供值。它们可以在两个轴上进行区分：a) 提供一个恒定值或从 @apidoc[RequestContext]中提取一个值  b) 提供一个单个值或一个元组。

>
 * @ref[extract](extract.md)
 * @ref[extractActorSystem](extractActorSystem.md)
 * @ref[extractDataBytes](extractDataBytes.md)
 * @ref[extractExecutionContext](extractExecutionContext.md)
 * @ref[extractLog](extractLog.md)
 * @ref[extractMatchedPath](extractMatchedPath.md)
 * @ref[extractMaterializer](extractMaterializer.md)
 * @ref[extractParserSettings](extractParserSettings.md)
 * @ref[extractRequestContext](extractRequestContext.md)
 * @ref[extractRequestEntity](extractRequestEntity.md)
 * @ref[extractRequest](extractRequest.md)
 * @ref[extractSettings](extractSettings.md)
 * @ref[extractStrictEntity](extractStrictEntity.md)
 * @ref[extractUnmatchedPath](extractUnmatchedPath.md)
 * @ref[extractUri](extractUri.md)
 * @ref[textract](textract.md)
 * @ref[provide](provide.md)
 * @ref[tprovide](tprovide.md)

<a id="request-transforming-directives"></a>
## 转换请求(上下文)

>
 * @ref[mapRequest](mapRequest.md)
 * @ref[mapRequestContext](mapRequestContext.md)
 * @ref[mapSettings](mapSettings.md)
 * @ref[mapUnmatchedPath](mapUnmatchedPath.md)
 * @ref[withExecutionContext](withExecutionContext.md)
 * @ref[withLog](withLog.md)
 * @ref[withMaterializer](withMaterializer.md)
 * @ref[withSettings](withSettings.md)
 * @ref[toStrictEntity](toStrictEntity.md)

<a id="response-transforming-directives"></a>
## 转变回应

这些指令允许挂钩(hook)到响应路径，并转换完整的响应或响应的一部分或拒绝列表：

>
 * @ref[mapResponse](mapResponse.md)
 * @ref[mapResponseEntity](mapResponseEntity.md)
 * @ref[mapResponseHeaders](mapResponseHeaders.md)

<a id="result-transformation-directives"></a>
## 转换RouteResult

这些指令允许转换内部路由的RouteResult。

>
 * @ref[cancelRejection](cancelRejection.md)
 * @ref[cancelRejections](cancelRejections.md)
 * @ref[mapRejections](mapRejections.md)
 * @ref[mapRouteResult](mapRouteResult.md)
 * @ref[mapRouteResultFuture](mapRouteResultFuture.md)
 * @ref[mapRouteResultPF](mapRouteResultPF.md)
 * @ref[mapRouteResultWith](mapRouteResultWith.md)
 * @ref[mapRouteResultWithPF](mapRouteResultWithPF.md)
 * @ref[recoverRejections](recoverRejections.md)
 * @ref[recoverRejectionsWith](recoverRejectionsWith.md)

## 其它

>
 * @ref[mapInnerRoute](mapInnerRoute.md)
 * @ref[pass](pass.md)

## 按字母顺序

@@toc { depth=1 }

@@@ index

* [cancelRejection](cancelRejection.md)
* [cancelRejections](cancelRejections.md)
* [extract](extract.md)
* [extractActorSystem](extractActorSystem.md)
* [extractDataBytes](extractDataBytes.md)
* [extractExecutionContext](extractExecutionContext.md)
* [extractLog](extractLog.md)
* [extractMatchedPath](extractMatchedPath.md)
* [extractMaterializer](extractMaterializer.md)
* [extractParserSettings](extractParserSettings.md)
* [extractRequest](extractRequest.md)
* [extractRequestContext](extractRequestContext.md)
* [extractRequestEntity](extractRequestEntity.md)
* [extractSettings](extractSettings.md)
* [extractStrictEntity](extractStrictEntity.md)
* [extractUnmatchedPath](extractUnmatchedPath.md)
* [extractUri](extractUri.md)
* [mapInnerRoute](mapInnerRoute.md)
* [mapRejections](mapRejections.md)
* [mapRequest](mapRequest.md)
* [mapRequestContext](mapRequestContext.md)
* [mapResponse](mapResponse.md)
* [mapResponseEntity](mapResponseEntity.md)
* [mapResponseHeaders](mapResponseHeaders.md)
* [mapRouteResult](mapRouteResult.md)
* [mapRouteResultFuture](mapRouteResultFuture.md)
* [mapRouteResultPF](mapRouteResultPF.md)
* [mapRouteResultWithPF](mapRouteResultWithPF.md)
* [mapRouteResultWith](mapRouteResultWith.md)
* [mapSettings](mapSettings.md)
* [mapUnmatchedPath](mapUnmatchedPath.md)
* [pass](pass.md)
* [provide](provide.md)
* [recoverRejections](recoverRejections.md)
* [recoverRejectionsWith](recoverRejectionsWith.md)
* [textract](textract.md)
* [toStrictEntity](toStrictEntity.md)
* [tprovide](tprovide.md)
* [withExecutionContext](withExecutionContext.md)
* [withLog](withLog.md)
* [withMaterializer](withMaterializer.md)
* [withSettings](withSettings.md)

@@@
