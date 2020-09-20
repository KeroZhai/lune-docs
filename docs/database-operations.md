# 数据库操作

框架目前支持的数据库是 MySQL。`app\lune\repository\Repository`类提供了一系列操作数据库的方法，使用前需要实例化。

在项目只有一个数据库的情况下，直接实例化一个默认的`Repository`类对象即可，或是采用依赖注入的方式，在接口方法中直接使用。例如：

``` php
<?php
namespace app\controller;
use app\lune\repository\Repository;

/**
 * @api
 * @mapping foo
 */
class FooController {
    /**
     * @get
     * @injected repository
     */
    function bar(Repository $repository) {
        // 直接使用
    }
}

```

## 保存数据

使用`$repository->save(object $obj)`方法可以直接将对象持久化到数据库中，例如有如下接口方法：

``` php
<?php
namespace app\controller;
use app\lune\response\StatusResult;
use app\lune\repository\Repository;

/**
 * 用户信息
 * @table user
 */
class UserInfo {
    public $name;
}

/**
 * @api
 * @mapping /user
 */
class UserController {
    /**
     * @post
     * @injected repository
     */
    function addUser(Repository $repository, UserInfo $userInfo) {
        $repository->save($userInfo);
        return StatusResult::success();
    }
}

```

在用户信息类上使用`@table`标记来指定对应的数据库表。由于自动构造 SQL 语句时，将会默认将驼峰风格的属性名转换为下划线分隔的全小写字母作为表中的字段名，如果数据库表字段命名风格不同，可以使用`@column`标记指定表中实际的字段名。例如：

``` php
<?php

/**
 * 用户信息
 * @table user
 */
class UserInfo {
    /**
     * @column Name
     */
    public $name;
}

```

## 更新数据

使用`$repository->update(object $obj)`方法来更新一条数据，类似于保存操作，但此时给定的对象中必须提供一个`$id`字段，用于定位到该条数据。

## 删除数据

使用`$repository->delete(string $tableName, int ...ids)`方法来根据 id 删除一条或多条数据。

## 查询数据

不同于前面的方法，查询方法需要自行编写 SQL 语句。

### 查询单条数据

使用`$repository->queryForOne(string $sql, $className, array $paramValues=null, array $paramTypes=null)`方法将查询出来的数据封装到指定类型的对象中。

例如按照姓名查询某个用户，并封装到一个用户类中：

``` php
$repository->queryForOne("SELECT * FROM user WHERE name = `$name`", UserInfo::class);
```

如果要使用预处理语句，则必须指定`$paramValues`，对应 SQL 语句中的占位符，`$paramTypes`用于指定对应的数据类型，缺省则均默认为`PDO::PARAM_STR`类型。所有支持的数据类型如下：

| 类型 | 含义  |
| :------------: | :------------: |
| `PDO::PARAM_BOOL`  |表示`bool`数据类型   |
| `PDO::PARAM_NULL` |表示`null`数据类型   |
| `PDO::PARAM_INT` |表示`int`数据类型  |
| `PDO::PARAM_STR` |表示字符串数据类型   |
| `PDO::PARAM_LOB` |表示大对象数据类型   |

则对应的调用方式则变成：

``` php
$repository->queryForOne("SELECT * FROM user WHERE name = ?", UserInfo::class, [$name], [\PDO::PARAM_STR]);
```

> 为防止 SQL 注入，建议**始终使用**预处理语句而非拼接 SQL。

### 查询多条数据

使用`$repository->queryForList(string $sql, $className, array $paramValues=null, array $paramTypes=null)`方法将每一条结果封装到指定类型的对象中，并以数组的形式返回。

### 查询分页数据

使用`$repository->queryForPage(string $sql, $page, $size, $className, array $paramValues=null, array $paramTypes=null)`将符合条件的数据封装到一个`PageInfo`类型的对象中，页码从 1 开始。

例如有以下接口：

``` php
<?php
namespace app\controller;
use app\lune\response\StatusResult;
use app\lune\repository\Repository;

/**
 * 用户信息
 */
class UserInfo {
    public $name;
}

/**
 * @api
 * @mapping /user
 */
class UserController {
    /**
     * @get
     * @injected $repository
     */
    function getUserPage(Repository $repository) {
        $pageInfo = $repository->queryForPage("SELECT * FROM user", 1, 10, UserInfo::class);
        return StatusResult::success("查询成功", $pageInfo);
    }
}

```

返回的数据格式如下：

``` json
{
    "status": 200,
    "success": true,
    "message": "查询成功",
    "data": {
        "content": [
            {
                "name": "Kero"
            },
            {
                "name": "Tom"
            },
            {
                "name": "Alice"
            }
        ],
        "empty": false,
        "last": true,
        "numberOfElements": 3,
        "page": 1,
        "size": 10,
        "totalElements": 3,
        "totalPages": 1
    }
}
```

`data`中即`PageInfo`对象包含的信息，各个属性的含义如下：

| 属性名  | 含义  |
| :------------: | :------------: |
| `content`  | 结果集  |
|  `empty` |  是否为空 |
| `last`  |  是否是最后一页 |
| `numberOfElements`  |  本页实际条数 |
| `page`  |  请求页码 |
| `size`  |  请求条数 |
|  `totalElements` | 总条数  |
| `totalPages`  |  总页数 |

### 查询PDOStatement

使用`$repository->execute(string $sql, array $paramValues=null, array $paramTypes=null)`方法会返回一个`PDOStatement`对象（关于`PDOStaement`，详见 [php PDO](https://www.php.net/manual/zh/class.pdostatement.php)），上述方法实际上都是基于此方法进行了拓展。

> 以上所有方法均会以抛出异常的形式报错。

## 事务

使用`$repository->beginTransaction()`方法开启事务，使用`$repository->commit()`和`$repository->rollback()`提交和回滚事务。

通过添加拦截器，可以实现在接口方法执行之前开启事务，接口方法执行之后提交事务，例如`app\core\controller\AbstractController`提供的两个拦截器：

``` php
/**
 * @before
 * @with @transactional
 * @injected repository
 */
function beginTransaction(Repository $repository) {
    $repository->beginTransaction();
}

/**
 * @after
 * @with @transactional
 * @injected repository
 */
function commitTransaction(Repository $repository) {
    $repository->commit();
}
```

当接口方法文档注释中包含了`@transactional`标记时，将会自动开启并提交事务。

?> 由于接口方法中未处理的异常被统一捕获后，后置拦截器将不会执行，即事务不会提交，因此无须进行显式的回滚。

!> 如果在接口方法中**捕获了异常而没有继续抛出**，则需要**手动调用**`$repository->rollback()`回滚事务。

## 多数据库

如果需要使用到多个数据库，则需要借助`app\core\repository\MysqlDB`类，它用于获取数据库连接（一个[`PDO`](https://www.php.net/manual/zh/class.pdo.php)类对象），该类维护一个数组用于暂存数据库连接。

!>  由于 php 并不会常驻内存，一次请求完成后相关的资源**均会被回收**，因此将数据库连接暂存于数组中仅仅是为了避免在**同一次请求**中重复创建相同的数据库连接。**数据库连接并不能在多个请求之间共享**。

`MysqlDB`类提供了以下三个静态方法：

| 方法定义   | 说明  |
| :------------: | :------------: |
| `MysqlDB::getDefaultConn(string $database=null)`  | 使用配置文件中的`[Datasource]`小节来获取数据库连接，默认使用配置文件中的数据库，也可单独指定   |
| `MysqlDB::getNamedConn(string $connKey, string $configSectionName, string $database=null)`  | 使用配置文件中自定义的小节来获取数据库连接  |
| `MysqlDB::newConn(string $address, string $user, string $password, string $database)`  | 使用给定的参数获取数据库连接  |

实际上，`app\core\repository\Repository`类的构造方法默认会调用`MysqlDB::getDefaultConn()`方法获取默认的数据库连接，也可以接收一个特定的数据库连接。

可以改变一个已有的`Repository`对象所使用的数据库连接，只需要调用`$repository->setConn($conn)`方法，传入一个新的数据库连接；或是自定义`Repository`的子类，然后在构造方法中设置数据库连接。如：

``` php
<?php
namespace app\controller;
use app\lune\repository\MysqlDB;
use app\lune\repository\Repository;

class AnotherRepository extends Repository {
    public function __construct() {
        $anotherConn = ...; // 通过 MysqlDB 获取新的数据库连接
        parent::__construct($anotherConn);
    }
}

/**
 * @api
 * @mapping foo
 */
class FooController {
    /**
     * @get
     * @injected repository anotherRepository
     */
    function bar(Repository $repository, AnotherRepository $anotherRepository) {
    }
}

```

!> 在改变一个已有的`Repository`对象所使用的数据库连接时，需要确保没有未完成的事务，即必须要首先调用`commit()`或`rollback()`方法。