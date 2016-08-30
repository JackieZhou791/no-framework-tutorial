[<< 上一节](08-dependency-injector.md) | [下一节 >>](10-dynamic-pages.md)

### 模板

模板引擎对于PHP来说意义并不大，因为PHP本身可以是模板渲染工具。不过它也有好久，比如过滤数据，分享程序逻辑和模板文件，使变量和HTML更好的结合。

最好读一读 [ircmaxell on templating](http://blog.ircmaxell.com/2012/12/on-templating.html).以及 [这个](http://chadminick.com/articles/simple-php-template-engine.html), 比较下对于模板引擎的不同观点。笔者本人对模板引擎没有太多的想法，你可以选择你喜欢的。

本节将使用 [Mustache](https://github.com/bobthecow/mustache.php)模板引擎. 

你也可以使用 [Twig](http://twig.sensiolabs.org/).

在Mustache类中 [engine class](https://github.com/bobthecow/mustache.php/blob/master/src/Mustache/Engine.php)你会发现，他并没有实现任何接口。

你可以在类型提示中直接使用Mustache 引擎类，但是这样会产生紧密耦合的问题。

换句话说，你的代码中所有使用这个模板引擎的地方都会直接调用Mustache包。假如我们想修改部分模板接口，就会出现问题。还比如我们想更换Twig引擎，然后加上部分自己的模板逻辑。这时我们只能回去修改所有和引擎调用相关的代码，这就是紧密耦合的问题。

为了松耦合，我们可以使用类型提示特性，将参数改为接口而不是具体的实现类。这时当我们要切换另外实现类的时候，只需要在新的类中实现相应的接口，这个类会自动注入到应用程序中。

我们可以使用适配器模式 [adapter pattern](http://en.wikipedia.org/wiki/Adapter_pattern)来处理这个问题。

首先我们可以定义接口，记住接口隔离原则[interface segregation principle](http://en.wikipedia.org/wiki/Interface_segregation_principle). 就是说确保接口方法细化，避免产生包含大量方法的接口。如果有必要，一个类可以继承多个接口。

这模板引擎这个实践中，我们需要的只是render这个方法。现在我们在src目录中创建Template目录，用放存放和模板相关的代码，然后创建Renderer.php接口：
```php
<?php

namespace Example\Template;

interface Renderer
{
    public function render($template, $data = []);
}
```

然后创建基于mustache的实现类，在同一个目录下，创建MustacheRenderer.php:

```php
<?php

namespace Example\Template;

use Mustache_Engine;

class MustacheRenderer implements Renderer
{
    private $engine;

    public function __construct(Mustache_Engine $engine)
    {
        $this->engine = $engine;
    }

    public function render($template, $data = [])
    {
        return $this->engine->render($template, $data);
    }
}
```

其实适配器模式就是这么简单。原始的实现类有很多方法，而新的类只需要实现适配器接口即可。
我们需要这个实现类加到Dependencies.php文件中，否则依赖注入器不知道接口的哪个实现类要被注入到处理逻辑中。

`$injector->alias('Example\Template\Renderer', 'Example\Template\MustacheRenderer');`


现在加Homepage控制器里，可以添加新的依赖了。

```php
<?php

namespace Example\Controllers;

use Http\Request;
use Http\Response;
use Example\Template\Renderer;

class Homepage
{
    private $request;
    private $response;
    private $renderer;

    public function __construct(
        Request $request, 
        Response $response,
        Renderer $renderer
    ) {
        $this->request = $request;
        $this->response = $response;
        $this->renderer = $renderer;
    }

...
```

我们还需要重写show方法，Mustache支持向械板中传递对象，我们先只传递简单的数组给模板。

```php
    public function show()
    {
        $data = [
            'name' => $this->request->getParameter('name', 'stranger'),
        ];
        $html = $this->renderer->render('Hello {{name}}', $data);
        $this->response->setContent($html);
    }
```

打开浏览器，看看是否工作正常。默认Mustache会使用字符串加载器，但我们需要的是模板文件。Mustache_Engine类构造函数中有关于加载器的配置。打开Dependencies.php,修改:

```php
$injector->define('Mustache_Engine', [
    ':options' => [
        'loader' => new Mustache_Loader_FilesystemLoader(dirname(__DIR__) . '/templates', [
            'extension' => '.html',
        ]),
    ],
]);
```

这里我们将mustache引擎默认的文件后缀从.mustache改为.html，这样是为了方便以后我们更像别的模板引擎。

在网站根目录下，我们创建templates目录，加入homepage.html文件：

```
<h1>Hello World</h1>
Hello {{ name }}
```

然后将Homepage 控制器里的render代码改为：

$html = $this->renderer->render('Homepage', $data);

打开浏览器看看吧！别忘了提交代码!
[<< 上一节](08-dependency-injector.md) | [下一节 >>](10-dynamic-pages.md)
