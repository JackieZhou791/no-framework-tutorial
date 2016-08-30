[<< 上一节](04-http.md) | [下一节 >>](06-dispatching-to-a-class.md)

### 路由

路由调度就是将不同的请求根据定义的规则调度到不同的处理程序。

上节中的代码URL请求只有一个返回结果，现在我们来增加路由功能。

本节中使用 [FastRoute](https://github.com/nikic/FastRoute) 来处理路由. 你可以选择你喜欢的组件，比如下列组件：

[symfony/Routing](https://github.com/symfony/Routing), [Aura.Router](https://github.com/auraphp/Aura.Router), [fuelphp/routing](https://github.com/fuelphp/routing), [Klein](https://github.com/chriso/klein.php)

你应该已经学会了如何安装组件了

现在将下面的代码加入到Bootstrap.php中替换上文中返回’Hello World’的地方

```php
$dispatcher = \FastRoute\simpleDispatcher(function (\FastRoute\RouteCollector $r) {
    $r->addRoute('GET', '/hello-world', function () {
        echo 'Hello World';
    });
    $r->addRoute('GET', '/another-route', function () {
        echo 'This works too';
    });
});

$routeInfo = $dispatcher->dispatch($request->getMethod(), $request->getPath());
switch ($routeInfo[0]) {
    case \FastRoute\Dispatcher::NOT_FOUND:
        $response->setContent('404 - Page not found');
        $response->setStatusCode(404);
        break;
    case \FastRoute\Dispatcher::METHOD_NOT_ALLOWED:
        $response->setContent('405 - Method not allowed');
        $response->setStatusCode(405);
        break;
    case \FastRoute\Dispatcher::FOUND:
        $handler = $routeInfo[1];
        $vars = $routeInfo[2];
        call_user_func($handler, $vars);
        break;
}
```

第一部分代码，你注册了应用程序可用的路由。第二部分调度方法被调用，进行switch语句中，如果路由找到了，相应的处理逻辑就会执行。

对是很小的项目，这样增加路由的方式可能够用。一旦项目很大，不可能在bootstrap文件中增加大量的路由，所以我们将它单独出一个文件。

在src目录下创建Routes.php文件。

```php
<?php

return [
    ['GET', '/hello-world', function () {
        echo 'Hello World';
    }],
    ['GET', '/another-route', function () {
        echo 'This works too';
    }],
];
```

Route集合也分离出来.

```php
$routeDefinitionCallback = function (\FastRoute\RouteCollector $r) {
    $routes = include('Routes.php');
    foreach ($routes as $route) {
        $r->addRoute($route[0], $route[1], $route[2]);
    }
};

$dispatcher = \FastRoute\simpleDispatcher($routeDefinitionCallback);
```

本节里加入了路由功能，改进了一点，但是路由调处还在Routes.php里，这不太理想，下节就来处理路由调度。

[<< 上一节](04-http.md) | [下一节 >>](06-dispatching-to-a-class.md)
