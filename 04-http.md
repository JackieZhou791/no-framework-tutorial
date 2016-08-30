[<< 上一节](03-error-handler.md) | [下一节 >>](05-router.md)

### HTTP

PHP提供了很多内置的特性用来方便的处理HTTP请求，比如 [超全局变量](http://php.net/manual/en/language.variables.superglobals.php)就包含了所有的请求信息。

如果你只是写一个脚本，可以很简单的使用全局变量来处理。但如果要写出干净，可维护，符合[SOLID](http://en.wikipedia.org/wiki/SOLID_%28object-oriented_design%29)标准的代码，就需要一个面向的对象的类来处理请求。

再强调一次，不用发明轮子，只需要安装一个你喜欢的第三方开发包即可。这里笔者使用了自己开发的一个组件[HTTP component](https://github.com/PatrickLouys/http) 。

你可以使用下面其它的组件:

[Symfony HttpFoundation](https://github.com/symfony/HttpFoundation), [Nette HTTP Component](https://github.com/nette/http), [Aura Web](https://github.com/auraphp/Aura.Web), [sabre/http](https://github.com/fruux/sabre-http)

安装组件只需修改composer.json并运行composer.update

```json
"require": {
    "php": ">=5.5.0",
    "filp/whoops": ">=1.1.2",
    "patricklouys/http": ">=1.1.0"
},
```

现在可以把下面的代码加入到Bootstrap.php的错误处理下面

```php
$request = new \Http\HttpRequest($_GET, $_POST, $_COOKIE, $_FILES, $_SERVER);
$response = new \Http\HttpResponse;
```

这里我们创建了Request和Response两个对象，你可以在其它类中获取请求对象，发送响应回浏览器。

当发送响应数据时，需要将下面的代码添加到Bootstrap.php的最后

```php
foreach ($response->getHeaders() as $header) {
    header($header, false);
}

echo $response->getContent();
```

这样才会将响应数据发送到浏览器端，要不然，Response对象只是存储了数据。各个不同的http 组件处理的方式大同小异，当你使用那些组件的时候记住这点。

header函数的第二个参数是false，作用是不覆盖已经存在的header变量。

现在Response发送了空的内容到浏览器，http状态码为200.你可以改成你想要的内容，像下面的代码：

```php
$content = '<h1>Hello World</h1>';
$response->setContent($content);
```

如果你返回404错误，下面的代码：

```php
$response->setContent('404 - Page not found');
$response->setStatusCode(404);
```

记住对象只用来存储数据，所以只是最后一个状态码会被返回给浏览器，就算你在对象中设置了很多次状态码。

下文中将会涉及组件的基本功能，同时，多看组件文档或代码有助于理解相关的内容。

[<< 上一节](03-error-handler.md) | [下一节 >>](05-router.md)
