# 响应对象

## 基础

作为示例，框架提供了`app\core\response\StatusResult`类来封装数据。它有以下四个属性：

| 属性名 | 类型 | 含义 |
| :------------: | :------------: | :------------: |
| `$status` |  `int` | 状态码 |
|  `$success` | `bool`  | 操作是否成功 |
|  `$message` | `string`  | 提示信息 |
| `$data`  | `mixed`  | 返回数据 |

!> **注意** 这里的状态码**并非**响应状态码，而应该是自定义的业务操作相关的状态码，这里默认以 200 表示操作成功。

该类的构造方法是私有的，即无法直接构造对象，但提供了三个静态方法：

| 方法定义 | 作用 |
| :------------: | :------------: |
| `success(string $message="Operation succeeded", $data=null)` | 返回操作成功时的结果，`$success`为`true`，`$staus`为`200` |
| ` error(string $message="Operation failed", $data=null)` | 返回操作失败时的结果，`$success`为`false`，`$staus`为`200`  |
| `status(int $status, string $message=null)` | 返回指定状态码的结果  |

默认情况下，所有接口的返回值均会以 json 格式输出，但也可以通过配置文件配置为返回原始数据。（关于配置文件，详见[配置](config)。）

直接在接口方法中返回`StatusResult::success()`，由于没有指定`$message`和`$data`的值，将会得到如下默认输出：

``` json
{
    "status": 200,
    "success": true,
    "message": "Operation succeeded"
}
```

?> 默认情况下，`null`值会被直接忽略，因此上面的输出中没有`$data`。

## 进阶

实际上，和请求对象类似，响应相关信息封装在`app\core\response\Response`对象中，作为接口方法的返回结果。

一般情况下（正如之前的例子中），并不需要用到此对象。此时，框架将会自动将方法的返回值封装到一个`Response`对象中。但有些时候，接口方法的返回值可能比较特殊，例如想要指定一个响应状态码或设置响应头，或者要[下载文件](upload-and-download#文件下载)，那就可以借助此对象来实现。

可以直接在接口方法中新建此对象，然后调用几个`setXXX()`方法进行配置，例如：

``` php
<?php

function foo() {
    $response = new Response();
    $response->setStatusCode(Response::NOT_FOUND);
    return $response;
}

```

或者是通过`Response::builder()`获取一个`ResponseBuilder`对象，然后快速构造一个简单的`Response`对象，例如：

``` php
<?php

function foo() {
    return Response::builder()->statusCode(Response::NOT_FOUND)->build();
}

```
或者是通过`Response::notFound()`方法，直接返回一个状态码为 404 的响应对象：

``` php
<?php

function foo() {
    return Response::notFound();
}

```

以上三种方式都会将接口的响应状态码设置为 404。

`Response`类所有的实例方法如下：

| 方法名 | 返回值类型 | 含义 |
| :------------: | :------------: | :------------: |
| `getStatusCode()` |  `int` | 返回响应状态码 |
|  `setStatusCode()` | 无  | 设置响应状态码，接收一个合法的状态码数值，推荐使用`Response::OK`等预定义的常量 |
|  `getHeaders()` | `array`  | 返回响应头 |
|  `getHeadersAsStringArray()` | `array`  | 以字符串数组的形式返回响应头 |
| `hasHeader()`  | `bool`  | 是否存在指定的响应头 |
| `getHeader()`  | `array`  | 返回指定的响应头的值，不存在则返回空数组|
| `getHeaderAsString()`  | `string`  | 以字符串形式（逗号分隔）返回指定的响应头的值，不存在则返回空字符串 |
| `setHeader(string $name, string ...$values)`  | 无  | 设置响应头 |
| `getBody()`  | `mixed`  | 返回响应 body |
| `setBody($body)`  | 无  | 设置响应 body |
| `builder()`  | `ResponseBuilder`  | 返回一个`ResponseBuilder`对象 |

`Response`类所有的静态方法如下：

| 方法名 | 返回值类型 | 含义 |
| :------------: | :------------: | :------------: |
| `ok($body = null)` |  `Response` | 返回一个状态码为 200 的响应对象 |
| `badRequest($body = null)` |  `Response` | 返回一个状态码为 400 的响应对象 |
| `notFound($body = null)` |  `Response` | 返回一个状态码为 404 的响应对象 |
| `error($body = null)` |  `Response` | 返回一个状态码为 500 的响应对象 |