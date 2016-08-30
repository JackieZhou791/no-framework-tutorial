[<< 上一节](06-dispatching-to-a-class.md) | [下一节 >>](08-dependency-injector.md)

### 控制反转

上节中你已经创建了控制器类并成功输出的返回结果。可别忘了，我们在第4节HTTP中引入了非常不错的HTTP抽象层，它还没有发挥作用。

控制反转是个不错的选择，它的意思是当需要使用某个类的时候，不是去创建一个对象，而是请求这个类。这就是Dependency injection依赖注入。

概念讲起来会有少少复杂，不用担心，根据下面的步骤我们来实现它。

修改Homepage.php:

```php
<?php

namespace Example\Controllers;

use Http\Response;

class Homepage
{
    private $response;

    public function __construct(Response $response)
    {
        $this->response = $response;
    }

    public function show()
    {
        $this->response->setContent('Hello World');
    }
}
```

注意你在文件头部引入了 [importing](http://php.net/manual/en/language.namespaces.importing.php) `Http\Response` 类. 这意味着可以直接使用Response类名，它会直接对应到完整的路径名。

构造函数中，你明确的请求了Http\Response。这样，Http\Response是一个接口，任何实现于这个接口的类都可以被注入。 参考 [type hinting](http://php.net/manual/en/language.oop5.typehinting.php) 和 [interfaces](http://php.net/manual/en/language.oop5.interfaces.php) 相关PHP文档.

现在代码会报错，因为你没有注入类。修改Bootstrap.php中路由调度的代码

```php
$class = new $className($response);
$class->$method($vars);
```

Http\Response 类实现于Http\Response接口，履行了接口的约束

现在应该工作正常了。但是如果你按照本节的做法，所有实例化的对象都会有相同的对象注入。这当然需要在下一节里改进。

[<< 上一节](06-dispatching-to-a-class.md) | [下一节 >>](08-dependency-injector.md)
