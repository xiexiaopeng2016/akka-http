# 预定义指令(按特质)

所有预定义的指令都组织成特质，它们构成总体 @apidoc[Directives] 特质的一部分。

<a id="request-directives"></a>
## 指令过滤或从请求中提取

@ref[MethodDirectives](method-directives/index.md)
:   基于请求方法进行筛选和提取。

@ref[HeaderDirectives](header-directives/index.md)
:   基于请求头进行过滤和提取。

@ref[PathDirectives](path-directives/index.md)
:   从请求URI路径中过滤和提取。

@ref[HostDirectives](host-directives/index.md)
:   基于目标主机进行过滤和提取。

@ref[ParameterDirectives](parameter-directives/index.md), @ref[FormFieldDirectives](form-field-directives/index.md)
:   根据查询参数或表单字段(Content-Type `application/x-www-form-urlencoded`或`multipart/form-data`)过滤和提取。

@ref[CodingDirectives](coding-directives/index.md)
:   过滤和解码压缩的请求内容。

@ref[Marshalling Directives](marshalling-directives/index.md)
:   提取请求实体。

@ref[SchemeDirectives](scheme-directives/index.md)
:   根据请求方案进行过滤和提取。

@ref[SecurityDirectives](security-directives/index.md)
:   处理来自请求的身份验证数据。

@ref[CookieDirectives](cookie-directives/index.md)
:   过滤和提取cookie。

@ref[BasicDirectives](basic-directives/index.md) and @ref[MiscDirectives](misc-directives/index.md)
:   处理请求属性的指令。

@ref[FileUploadDirectives](file-upload-directives/index.md)
:   处理文件上传。

<a id="response-directives"></a>
## 创建或转换响应的指令

@ref[CacheConditionDirectives](cache-condition-directives/index.md)
:   支持条件请求(`304 Not Modified`响应)。

@ref[CachingDirectives](caching-directives/index.md)
:   支持缓存昂贵的操作。

@ref[CookieDirectives](cookie-directives/index.md)
:   设置、修改或删除cookie。

@ref[CodingDirectives](coding-directives/index.md)
:   压缩响应。

@ref[FileAndResourceDirectives](file-and-resource-directives/index.md)
:   从文件和资源中交付响应。

@ref[RangeDirectives](range-directives/index.md)
:   支持范围请求(`206 Partial Content`响应)。

@ref[RespondWithDirectives](respond-with-directives/index.md)
:   改变的响应特性。

@ref[RouteDirectives](route-directives/index.md)
:   使用响应完成或拒绝请求。

@ref[BasicDirectives](basic-directives/index.md) and @ref[MiscDirectives](misc-directives/index.md)
:   指令处理或转换响应属性。

@ref[TimeoutDirectives](timeout-directives/index.md)
:   配置请求超时和自动超时响应。

## 按特质的预定义指令列表

@@toc { depth=1 }

@@@ index

* [basic-directives/index](basic-directives/index.md)
* [cache-condition-directives/index](cache-condition-directives/index.md)
* [caching-directives/index](caching-directives/index.md)
* [coding-directives/index](coding-directives/index.md)
* [cookie-directives/index](cookie-directives/index.md)
* [debugging-directives/index](debugging-directives/index.md)
* [execution-directives/index](execution-directives/index.md)
* [file-and-resource-directives/index](file-and-resource-directives/index.md)
* [file-upload-directives/index](file-upload-directives/index.md)
* [form-field-directives/index](form-field-directives/index.md)
* [future-directives/index](future-directives/index.md)
* [header-directives/index](header-directives/index.md)
* [host-directives/index](host-directives/index.md)
* [marshalling-directives/index](marshalling-directives/index.md)
* [method-directives/index](method-directives/index.md)
* [misc-directives/index](misc-directives/index.md)
* [parameter-directives/index](parameter-directives/index.md)
* [path-directives/index](path-directives/index.md)
* [range-directives/index](range-directives/index.md)
* [respond-with-directives/index](respond-with-directives/index.md)
* [route-directives/index](route-directives/index.md)
* [scheme-directives/index](scheme-directives/index.md)
* [security-directives/index](security-directives/index.md)
* [websocket-directives/index](websocket-directives/index.md)
* [timeout-directives/index](timeout-directives/index.md)

@@@
