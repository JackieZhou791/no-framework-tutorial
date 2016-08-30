[<< 上一节](07-inversion-of-control.md) | [下一节 >>](09-templating.md)

### 依赖注入器 

依赖注入器的目的是处理依赖问题，确保当类实例化的时候，正确的对象能被注入。

老实讲，笔者只推荐使用: [Auryn](https://github.com/rdlowrey/Auryn). 遗憾的是基本所有的替代器都在使用 [service locator antipattern](http://blog.ploeh.dk/2010/02/03/ServiceLocatorisanAnti-Pattern/).

安装完Auryn后在src目录中创建Dependencies.php

```php
<?php

$injector = new \Auryn\Injector;

$injector->alias('Http\Request', 'Http\HttpRequest');
$injector->share('Http\HttpRequest');
$injector->define('Http\HttpRequest', [
    ':get' => $_GET,
    ':post' => $_POST,
    ':cookies' => $_COOKIE,
    ':files' => $_FILES,
    ':server' => $_SERVER,
]);

$injector->alias('Http\Response', 'Http\HttpResponse');
$injector->share('Http\HttpResponse');

return $injector;
```

继续下去之前，确保你已经理解了alias,share,define这意义和用法。请前往Auryn文档了解更多。

Share Http对象可以做到http对象的唯一性，避免出现同一类被实例化多次，出现http对象的不一致。

Alias的可以用接口代码类来进行类型映射，这样有助于继承类代码的维护，不用去修改原始的类实现。

现在Bootstrap.php需要修改了，在$request和$response被实例化前，将他们注册到injector中确保所有地方的实例的唯一性。

```php
$injector = include('Dependencies.php');

$request = $injector->make('Http\HttpRequest');
$response = $injector->make('Http\HttpResponse');
```
路由调度也需要修改

将
```php
$class = new $className($response);
$class->$method($vars);
```

改为

```php
$class = $injector->make($className);
$class->$method($vars);
```

现在所有的控制器依赖就由Auryn来自动处理了，Homepage.php改为：
```php
<?php

namespace Example\Controllers;

use Http\Request;
use Http\Response;

class Homepage
{
    private $request;
    private $response;

    public function __construct(Request $request, Response $response)
    {
        $this->request = $request;
        $this->response = $response;
    }

    public function show()
    {
        $content = '<h1>Hello World</h1>';
        $content .= 'Hello ' . $this->request->getParameter('name', 'stranger');
        $this->response->setContent($content);
    }
}
```

现在你看到Homepage类有两个依赖注入了，试着访问一个带GET参数的url
http://localhost:8000/?name=Arthur%20Dent .

祝贺你，现在这个应用程序的基础基本被打好了。

[<< 上一节](07-inversion-of-control.md) | [下一节 >>](09-templating.md)
