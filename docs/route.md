# 路由

接收到请求`[GET] /api/foo/bar`后，定位到`FooController`的`bar()`方法的过程，称为路由。

## 使用标记来配置路由

在类的文档注释中使用`@mapping`标记来配置第一级 URL 映射，即 URL 与接口类的映射。

在方法的文档注释中使用`@get`、`@post`、`@put`、`@patch`和`@delete`标记（分别对应五种请求方式）来配置第二级 URL 映射，即 URL 与接口方法的映射。

接口类的路由配置是必须的，且必须有指定的值。接口方法的路由配置不一定需要指定值，如果为空，则默认映射为`/`。例如：

``` php
<?php
namespace app\controller;

/**
 * @api
 * @mapping foo
 */
class FooController {
    /**
     * @get
     */
    function bar() {
        return "Hello world";
    }
}

```
发送请求`[GET] /api/foo`，得到输出：

``` json
"Hello world"
```

同时，由于标记指定了该方法需要使用`GET`请求，如果使用`POST`请求，将会返回 HTTP 状态码 405，表示请求方式不匹配。

## 路由标记的值

一般情况下，路由标记的值是普通的字符串，可以以`/`开头，也可以省略，例如`@get /foo`和`@get foo`都是合法的配置。

如果要使用 RESTful 风格的路由，例如`/user/1`的形式，则有所不同。例如：

``` php
<?php
namespace app\controller;

/**
 * @api
 * @mapping /user
 */
class UserController {
    /**
     * @get /{id}
     */
    function getUserById(int $id) {
        return $id;
    }
}

```

此时URL中的`/1`将会匹配到配置的`/{id}`，并将值`1`绑定到参数`$id`，传入方法中。

?> 关于参数绑定，详见[参数绑定](binding#参数绑定)

发送请求`[GET] /api/user/1 `，将会得到输出：

``` json
1
```