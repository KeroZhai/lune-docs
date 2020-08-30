# 过滤器与拦截器

过滤器和拦截器均是定义在接口中的方法，同样通过标记来配置。

## 过滤器

过滤器可以定义多个，会在接口方法被执行之前**按照顺序调用**，用来对请求数据预处理，例如将请求数据强制转换为utf-8编码。

使用`@filter`标记定义一个过滤器，可以定义名为`&$__metadata__`的形参，用于接收请求数据的**引用**。

!> 请求数据以一个关联数组的形式提供，如果不加`&`，则取到的数据只是值的**一份拷贝**，对其改变不会影响到实际的数据。

例如：

``` php
<?php
namespace app\controller;

/**
 * @api
 * @mapping /user
 */
class UserController {
    /**
     * @filter
     */
    function doSomeFilterStuff(&$__metadata__) {
        // 在请求数据中新增id=1
        $__metadata__["id"] = 1;
    }

    /**
     * @get
     */
    function getUserById(int $id) {
        return $id;
    }
}

```

发送请求`[GET] /api/user`，即使没有携带任何的请求数据，但因为在过滤器中改变了请求数据，并最终进入接口方法中，因此同样可以得到输出 1。

## 拦截器

使用`@before`定义一个前置拦截器，前置拦截器同样会根据配置在接口方法被执行之前按顺序调用，但在过滤器方法之后。前置拦截器用于在接口方法执行之前做一些操作，例如判断是否登录等。使用`@after`定义一个后置拦截器，后置拦截器在接口方法之后调用。

任意一个前置拦截器方法中如果有返回值，将不会执行后续方法，以达到拦截请求的效果。后置拦截器的返回值则会始终被忽略。例如：

``` php
<?php
namespace app\controller;

/**
 * @api
 * @mapping /user
 */
class UserController {
    /**
     * @before
     */
    function intercept($__metadata__) {
        return "Intercepted successfully";
    }

    /**
     * @get
     */
    function sayHello() {
        return "Hello";
    }

    /**
     * @after
     */
    function post() {
        return "Post";
    }
}

```

发送请求`[GET] /user`，将会得到输出：

``` json
"Intercepted successfully"
```

前置拦截器拦截了请求，接口方法和后置拦截器方法均没有被执行。

两种拦截器方法都可以定义名为`$__method__`和`$__metadata__`的形参，分别用于接收接口方法对应`ReflectionMethod`对象和请求数据。

!> **注意**：不同于过滤器方法，传入拦截器方法的`$__metadata__`并不是引用，因此如果加上`&`将会报错。

默认情况下，拦截器方法始终会被执行，但可以使用`@with`标记指定一个标记名，当且仅当指定的接口方法的文档指数中存在指定的标记时，拦截器方法才会被执行。例如：

``` php
<?php
namespace app\controller;

/**
 * @api
 * @mapping /user
 */
class UserController {
    /**
     * @before
     * @with @someTag
     */
    function intercept($__metadata__) {
        return "Intercepted successfully";
    }

    /**
     * @someTag
     * @get /withTag
     */
    function withTag() {
        return "with tag";
    }

    /**
     * @get /withoutTag
     */
    function withoutTag() {
        return "Without tag"
    }
}

```

发送请求`[GET] /api/user/withTag`，则前置拦截器方法将会执行，得到输出：

``` json
"Intercepted successfully"
```

发送请求`[GET] /api/user/withoutTag`，将会得到输出：

``` json
"Without tag"
```

## 示例

基于拦截器，Lune 提供了一个简单的认证和鉴权功能。在`app\core\controller\AbstractController`类中分别定义了验证登录和用户角色的拦截器方法，使用时需要将接口类继承此类，例如：

``` php
<?php
namespace app\controller;
use app\core\controller\AbstractController;

/**
 * @api
 * @mapping /user
 */
class UserController extends AbstractController {
    /**
     * @get
     * @requiresAuthentication
     * @requiresRole admin
     */
    function getUser() {
    }
}

```

然后通过`app\core\util\AuthUtils`类来管理登录状态，它包含以下方法：

| 方法定义   | 说明  |
| :------------: | :------------: |
| `AuthUtils::login($user)`  | 传入一个表示用户的自定义对象，以该对象构造`AuthUser`对象，放入 Session 中并返回   |
| `AuthUtils::isLoggedIn()`  |判断用户是否登录  |
| `AuthUtils::logout()`  | 从 Session 中删除`AuthUser`对象  |
| `AuthUtils::getAuthUser()`  | 返回当前的`AuthUser`对象，如果还未登录将会抛出异常  |
| `AuthUtils::setRoles(string ...$roles)`  | 设置当前用户的角色，可以多个 |
| `AuthUtils::getRoles()`  | 获取当前用户的角色 |

请求`[GET] api/user`时，如果用户还未登录，将会返回：

``` json
{
    "status": 401
}
```

如果用户已登录，但没有角色`admin`，则返回：

``` json
{
    "status": 403
}
```

拦截器的另一个应用示例是[事务](database-operations#事务)。