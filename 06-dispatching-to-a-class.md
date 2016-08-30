[<< 上一节](05-router.md) | [下一节 >>](07-inversion-of-control.md)

### 路由调度

本系列文章中我们不去实现 [MVC (Model-View-Controller)](http://martinfowler.com/eaaCatalog/modelViewController.html)。 对PHP来说，MVC模式也很难真正的实现，就算实现，也和原本的MVC定义有很大出入。 可以去阅读 [A Beginner's Guide To MVC](http://blog.ircmaxell.com/2014/11/a-beginners-guide-to-mvc-for-web.html) 相关文章了解更多。

所以暂时忘掉MVC，考虑下关注隔离“Separation of concerns”[separation of concerns](http://en.wikipedia.org/wiki/Separation_of_concerns).

我们先定义类的描述语去处理请求。这里我们使用控制器”Controllers”来处理，大家多半用框架里了解到这个术语。当然也可以定义为Handlers。

我们在src目录下创建一个Controllers子目录，在这个子目录中，我们创建Homepage.php

```php
<?php

namespace Example\Controllers;

class Homepage
{
    public function show()
    {
        echo 'Hello World';
    }
}
```

如果文件名和类名相同，自动加载器会自动加载该文件。在本系统文章的开始我们定义了Example为应用程序的根命名空间，他是批向src/目录的。

现在我们来修改hello world路由让他能调用我们创建的类，而不是执行闭包函数。修改Routes.php文件：

```php
return [
    ['GET', '/', ['Example\Controllers\Homepage', 'show']],
];
```

第三个参数从闭包变成数组，数组的第一个值是包含命名空间完整路径的类名，第二个是你想调用的方法名

为了让上面的的改动生效，你需要给Bootstrap.php的路由部分做一点小的重构

```php
case \FastRoute\Dispatcher::FOUND:
    $className = $routeInfo[1][0];
    $method = $routeInfo[1][1];
    $vars = $routeInfo[2];
    
    $class = new $className;
    $class->$method($vars);
    break;
```

原来是执行闭包函数，现在的处理是实例化对象，并调用他的方法

现在访问 http://localhost:8000/ 测试下是否正常，如果有误，请检查上述步骤。

[<< 上一节](05-router.md) | [下一节 >>](07-inversion-of-control.md)