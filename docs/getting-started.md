# 起步

## 安装

只需要把框架下载下来，放到一个合适的目录，就算完成*“安装”*。

## 目录结构

一个使用 Lune 的项目目录结构应当如下：

```
APP_ROOT（项目逻辑根目录）
 |-- conf
 |    ╰-- config.ini（配置文件）
 |-- controller（默认接口类存放目录）
 |-- lune（Lune 目录）
 |-- index（项目实际主目录）
 |    |-- static（静态资源存放目录）
 |    |-- .htaccess（URL 重写规则）
 |    ╰-- index.php（项目入口文件）
 ╰-- view（页面文件存放目录）
```

## 一些准备工作 

虽然在逻辑上，应用的根目录应该是`APP_ROOT`，但考虑到安全性，**应该只有`APP_ROOT/index`目录能够被直接访问**，也即对 Apache 来说，应用的实际的根目录应该是`APP_ROOT/index`。

由于涉及到了 URL 重写，所以需要先修改 Apache 的配置文件，将与项目对应的`<Directory></Directory>`中的`AllowOverride`由`None`改为`All`。以虚拟主机的配置为例：

```
NameVirtualHost *:PORT
<VirtualHost *:PORT>
    DocumentRoot "APP_ROOT/index"
    DirectoryIndex index.php
    <Directory "APP_ROOT/index">
        Options FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

配置完成后，直接在浏览器中访问`localhost:PORT`或`localhost:PORT/welcome`，将会显示欢迎页面。

## 命名空间规范

Lune 的自动加载机制遵循了 PSR-4 规范，简而言之，需要遵守以下约定：

* 一个文件中只有一个类，且类名必须和文件名保持一致，大小写敏感
* 类所在的[命名空间](https://www.php.net/manual/zh/language.namespaces.rationale.php)需必须和实际的目录对应，以`app`开头，对应项目根目录（如无特别说明，均指代逻辑根目录）

例如：

| 类名 | 命名空间 | 实际路径 |
| :------------: | :------------: | :------------: |
| `UserController` |  `app\controller` | APP_ROOT/controller/UserController.php |
|  `UserPO` | `app\po`  | APP_ROOT/po/UserPO.php |

## 什么是标记？

类似于 Javadoc，PHP 同样支持文档注释。文档注释分为类文档注释和方法文档注释两种，用于对类或方法进行描述。例如：

``` php
/**
 * 返回传入的字符串
 * 
 * @param value 要返回的字符串
 * @return string 原始字符串
 */
function getString(string $value) {
    return $value;
}
```

其中的`@param`和`@return`被称为文档注释中的标记（Tag），如果 IDE 支持，你将会看到它们拥有代码高亮效果。然而，除了一些预定义的标记以外，其余的部分均会被视作是普通的内容，即使把它们也写成`@XX`的标记形式。但幸好，PHP 的反射机制能够让我们获取到类或方法上面的文档注释，从而让自定义的标记发挥作用。

?> 尽管对于我们来说，自定义的标记是有特殊含义的，但它们并不支持代码高亮，这是因为对于 IDE（或更确切地，代码高亮功能）来说，自定义标记仍然只是普通的文本。
