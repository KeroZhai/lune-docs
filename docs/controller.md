# 接口

接口（也称控制器，即 Controller）用于处理请求。

默认情况下，Lune 会从根目录下的 controller 目录中扫描接口类，对应的命名空间是`app\controller`。如果想将接口类编写在其他目录或是嵌套的子目录中，则需要在[配置文件](config)中配置路径匹配模式`scan-pattern`，该模式规则与`glob()`方法保持一致，参见 [PHP: glob](https://www.php.net/manual/zh/function.glob.php)。

例如采用以下的目录结构存放接口类文件（省略其他目录）：

```
APP_ROOT（项目逻辑根目录）
 |-- controller（默认接口类存放目录）
 |    ╰-- sub（子目录）
 ╰-- another（另一个存放目录）
```

那么`scan-pattern`应该配置为：`/controller/*, /controller/sub/*, /another/*`。

接口类本身不需要继承任何类或实现任何接口，但也可以继承`app\lune\controller\AbstractController`类以使用其中提供的拦截器方法，或是继承自定义的父类以进行统一配置。

## 分类

接口分为两种，一种用来返回数据，一种用来返回页面。

?> 通过接口来返回页面的目的是**限制可访问性**，但如果不需要限制页面文件的访问，也可以直接把页面文件中放到`index/static`目录下，然后在浏览器中通过相对路径访问页面文件。

在类文档注释中使用`@api`标记表明它是一个数据接口，使用路由标记来配置 URL 映射关系。（关于路由，具体的介绍与使用参见：[路由](route#路由)。）

所有的数据接口请求映射都会默认加上`/api`的前缀，例如有如下接口类：

``` php
<?php
namespace app\controller;

/**
 * @api
 * @mapping foo
 */
class FooController {
    /**
     * @get bar
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

在类文档注释中使用`@view`标记表明它是一个页面接口。在`controller`目录下默认有一个`WelcomeController.php`，它用来返回项目的初始欢迎页：

``` php
<?php
namespace app\controller;

/**
 * @view
 */
class WelcomeController {
    /**
     * @get
     */
    function showWelcome() {
        return "welcome.html";
    }
    /**
     * @get welcome
     */
    function showWelcome2() {
        return "welcome.html";
    }
}

```

可以看到，对于页面接口来说，类标记`@mapping`不是必须的，缺省会映射到`/`，但也可以额外指定以作区分。接口方法实际返回的页面文件取决于其返回值，返回值是页面文件相对于`view`目录的路径，可以是多级，但必须以`.html`或`.php`结尾。

!> 不同于数据接口，请求页面接口时**不需要**加上`/api`前缀。

实际上，数据接口和页面接口对返回值的处理没有区别，因此数据接口也可以返回页面，页面接口也可以返回数据。但一般情况下，二者应当各司其职，以便分别进行开发和维护。
