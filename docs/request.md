# 请求对象

框架将请求的相关信息封装到了`app\core\request\Request`类对象中，可以通过依赖注入的方式获取到。例如：

``` php
<?php
namespace app\controller;

use app\core\request\Request;

/**
 * @api
 * @mapping foo
 */
class FooController {
    /**
     * @get
     * @injected request
     */
    function bar(Request $request) {
        return $request->getCookieParams();
    }
}

```

请求`[GET] /api/foo`将会输出：

``` json
{
    "PHPSESSID": "5vehf1jib5r9c51apgr74kk38u"
}
```

除了获取 Cookies 以外，还提供了以下几个方法：

| 方法名 | 返回值类型 | 含义 |
| :------------: | :------------: | :------------: |
| `getUri()` |  `string` | 返回 URI(relative) |
|  `getMethod()` | `string`  | 返回请求方式 |
|  `getServerParams()` | `array`  | 返回`$_SEREVER`的**一份拷贝** |
| `getHeaders()`  | `array`  | 返回请求头 |
| `getQueryParams()`  | `array`  | 返回 URI 中的查询参数 |
| `getParsedBody()`  | `array`  | 返回解析后的 body 中的数据，支持 form-data、x-www-form-urlencoded 或 JSON |
| `getData()`  | `array`  | 返回所有的请求数据 |
| `getCookieParams()`  | `array`  | 返回 Cookies |
| `getUploadedFiles()`  | `UploadedFiles`  | 返回包含上传的所有文件的对象，是一个`UploadedFile`的集合，详见[文件上传](#文件上传) |