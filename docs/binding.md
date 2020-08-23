# 参数绑定

接口方法定义了形参，也即该接口方法希望接收的数据，参数绑定的过程就是取出对应的请求数据，传入方法中。

目前支持自动绑定以下四种形式的请求数据：

* URL 参数
* RESTful 风格
* form-data
* JSON

前两种形式的请求数据均直接体现在 URL 中，而后两种则是在请求 body 中。

> 除了普通数据以外，同样支持[文件上传](#文件上传)。

## URL 参数

URL 参数，也即`/user?id=1&name=kero`的形式，例如有如下接口：

``` php
<?php
namespace app\controller;

/**
 * @Api
 * @Mapping /user
 */
class UserController {
    /**
     * @GetMapping
     */
    function getUserById(int $id, string $name) {
        return "$id $name";
    }
}

```

发送请求`[GET] /api/user?id=1&name=kero`，将会得到以下输出：

``` json
"1 kero"
```

## RESTful 风格

前面提到，RESTful 风格的 URL 类似于`/user/1`，其中的`1`就是携带的数据，可以理解为是一种特殊的 URL 参数。

URL 参数可以与 form-data 或 JSON 数据共存，例如有如下接口：

!> **注意** 如果有同名的参数，URL 参数将会被 body 中的参数覆盖。

``` php
<?php
namespace app\controller;

/**
 * @Api
 * @Mapping /user
 */
class UserController {
    /**
     * @PostMapping /{id}
     */
    function getUserByIdAndName(int $id, string $name) {
        return "$id: $name";
    }
}

```

?> 在**[RFC7231](https://datatracker.ietf.org/doc/rfc7231/)**中关于`GET`请求的定义中提到，`GET`请求中的 body 数据是没有意义的，并且会被忽略，因此这里不能使用`GET`请求。

在 form-data 中添加`name=kero`的表项，或是使用 JSON 格式的数据：

``` json
{
    "name": "kero"
}
```

然后发送请求`[POST] /api/user/1`，都可以得到输出：

``` json
"1: kero"
```

两种形式的 URL 参数本身也是可以同时存在的，并且值会自动组合，即使看起来会比较奇怪，例如请求为`[GET] /api/user/1?id=2,3`，而接口如下：

``` php
<?php
namespace app\controller;

/**
 * @Api
 * @Mapping /user
 */
class UserController {
    /**
     * @GetMapping /{id}
     */
    function getUserByIdAndName(string $id) {
        return explode(',', $id);
    }
}

```

此时`$id`的值为`"1,2,3"`则输出为：

``` json
[
    "1",
    "2",
    "3"
]
```

### 使用对象接收

在之前的例子中，接口方法中的形参都是和请求数据一一对应的，大部分情况下，要发送的数据可能很多，例如在新增用户信息时，需要提交用户的姓名、性别、手机号、邮箱、详细地址等等数据，此时形参的定义无疑很麻烦，因此，可以定义一个类，例如`UserInfo`，作为一个容器来统一接收上面的数据。如：

``` php
<?php
namespace app\controller;
use app\core\controller\StatusResult;

/**
 * 用户信息
 */
class UserInfo {
    public $name;
    public $gender;
    public $phone;
    public $mail;
    public $address;
}

/**
 * @Api
 * @Mapping /user
 */
class UserController {
    /**
     * @PostMapping
     */
    function addUser(UserInfo $userInfo) {
        return StatusResult::("新增成功", $userInfo);
    }
}

```

?> 为方便说明，`UserInfo`类和`UserController`声明在了同一个文件中，但在实际开发中**不推荐**这样做，应当一个类对应一个文件（详见 PSR 规范）。

这里省略了实际的数据库操作，只是简单地将用户信息返回。`StatusResult`是一个用于封装返回值的类，详见[响应对象](response)。

发送请求`[POST]: /api/user`并携带如下的JSON数据：

``` json
{
    "name": "kero",
    "gender": "male",
    "phone": "123465",
    "mail": "xxx@gmail.com",
    "address": "chengdu"
}
```

得到输出：

``` json
{
    "status": 200,
    "success": true,
    "message": "新增成功",
    "data": {
        "name": "kero",
        "gender": "male",
        "phone": "123465",
        "mail": "xxx@gmail.com",
        "address": "chengdu"
    }
}
```

可以看到 JSON 中的数据自动绑定到了参数`$userInfo`的属性中，使用时直接以`$userInfo->name`的形式访问即可。

指定了自定义类型的参数也可以和一般类型的参数共存，例如将上面的接口方法改为：

``` php
/**
 * @PostMapping
 */
function getUserByIdAndName(int $id, UserInfo $userInfo) {
    return StatusResult::("新增成功", ["id" => $id, "userInfo" => $userInfo]);
}
```

同时改动一下 JSON 数据的内容：

``` json
{
    "userInfo": {
        "name": "kero",
        "gender": "male",
        "phone": "123465",
        "mail": "xxx@gmail.com",
        "address": "chengdu"
    },
    "id": 1
}
```

将会得到输出：

``` json
{
    "status": 200,
    "success": true,
    "message": "新增成功",
    "data": {
        "id": 1,
        "userInfo": {
            "name": "kero",
            "gender": "male",
            "phone": "123465",
            "mail": "xxx@gmail.com",
            "address": "chengdu"
        }
    }
}
```

可以看出，参数绑定和接口方法中形参定义的顺序是**无关的**。