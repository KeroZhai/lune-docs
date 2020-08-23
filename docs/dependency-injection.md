# 依赖注入

调用接口方法时，请求数据的参数绑定是自动进行的。如果有别的**自定义类型**参数希望自动传入方法中，可以借助依赖注入来实现（例如使用默认配置的数据库操作对象，参见[数据库操作](#database-operations)）。

?> 依赖注入（DI，Dependency Injection）或控制反转（IoC，Inversion of Control），通俗来讲就是**使用**（依赖）某一个对象时，不显式地直接创建它，而是交由其他部分来完成它的实例化。

目前只有接口类的构造器方法、接口方法和拦截器方法支持依赖注入。依赖注入的优先级**大于**请求参数绑定的优先级，如果一个参数被配置为了依赖注入，即使请求参数中有同名的数据也会被忽略。

!> 依赖注入的对象是**单例**的，这意味着在一次请求过程中，对于同一种类型将始终注入同一个对象。

在方法文档注释中使用`@Injected`标记来标识需要自动注入的参数，以空格分隔，与参数的实际定义**顺序无关**。例如：

``` php
<?php
namespace app\controller;

class A {
    public $name = "A";
}

class B {
    public $name = "B";
}

/**
 * @Api
 * @Mapping foo
 */
class FooController {

    private $a;

    /**
     * 构造方法注入
     * @Injected a
     */
    function __construct(A $a) {
        $this->a = $a;
    }

    /**
     * 接口方法直接注入
     * @GetMapping
     * @Injected b
     */
    function testInjection(B $b) {
        $a = $this->a;
        return "$a->name, $b->name";
    }
}

```

请求`[GET] /api/foo`将会输出：

``` json
"A, B"
```

!> 依赖注入的类的构造方法不能是`private`的，除了需要注入的参数以外，其他参数必须提供默认值。