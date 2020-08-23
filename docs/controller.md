# 接口

Lune 使用[PHP 命名空间](https://www.runoob.com/php/php-namespace.html)组织代码，接口（也称控制器，即 Controller）用于处理请求，编写在根目录下的`controller`中，定义在命名空间`app\controller`下。

接口类的类名**必须**和文件名保持一致，例如`UserController`类所在的文件名必须是`UserController.php`。接口类本身不需要继承任何类或实现任何接口，但也可以继承`app\core\AbstractController`类以使用其中提供的拦截器方法，或是继承自定义的父类以进行统一配置。由于配置了自动加载，在接口中引入框架提供的类时不需要根据文件实际路径写`require`语句。

接口分为两种，一种用来返回数据，一种用来返回页面。通过接口来返回页面的目的是**限制可访问性**，但如果不需要限制页面文件的访问，也可以直接把页面文件中放到`index/static`目录下，然后在浏览器中通过相对路径访问页面文件。

在类文档注释中使用`@Api`标记表明它是一个数据接口，使用路由标记来配置 URL 映射关系。（关于路由，具体的介绍与使用参见：[路由](route#路由)。）

所有的数据接口请求映射都会默认加上`/api`的前缀，例如有如下接口类：

``` php
<?php
namespace app\controller;

/**
 * @Api
 * @Mapping foo
 */
class FooController {
    /**
     * @GetMapping bar
     */
    function bar() {
        return "Hello world";
    }
}

```

发送请求`[GET] /api/foo/bar`时（例如直接在浏览器中访问），则会得到如下输出：

``` json
"Hello world"
```

在类文档注释中使用`@View`标记表明它是一个页面接口。在`controller`目录下默认有一个`WelcomeController.php`，它用来返回项目的初始欢迎页：

``` php
<?php
namespace app\controller;

/**
 * @View
 */
class WelcomeController {
    /**
     * @GetMapping
     */
    function showWelcome() {
        return "welcome.html";
    }
    /**
     * @GetMapping welcome
     */
    function showWelcome2() {
        return "welcome.html";
    }
}

```

对于页面接口来说，类标记`@Mapping`不是必须的，缺省会映射到`/`，也可以额外指定以作区分。接口方法实际返回的页面文件取决于其返回值，返回值是页面文件相对于`view`目录的路径，可以是多级，但必须以`.html`或`.php`结尾。请求页面接口时**不需要**加上`/api`前缀。

?> 实际上，数据接口和页面接口对返回值的处理没有区别，因此数据接口也可以返回页面，页面接口也可以返回数据。一般情况下，二者应当各司其职，以便分别进行开发和维护。
