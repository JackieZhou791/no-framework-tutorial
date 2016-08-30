[<< 上一节](09-templating.md) | [下一节 >>](11-page-menu.md)

### 动态页面

到现在为止，我们已经可以创建没有多少功能的静态页面了。“Hello World”实在没有太大用处，现在我们给应用程序加一个有实际功能的逻辑。

第一个功能是根据[MD](http://en.wikipedia.org/wiki/Markdown) 文件生成动态页面。

先创建一个Page控制器：

```php
<?php

namespace Example\Controllers;

class Page
{
    public function show($params)
    {
        var_dump($params);
    }
}
```
然后添加下面的路由：

```php
['GET', '/{slug}', ['Example\Controllers\Page', 'show']],
```

尝试访问下http://localhost:8000/test，或http://localhost:8000/hello. Page控制器会被调用，$params数组会接收网站中的slug参数。

因为还没有使用数据库，我们先在网站根目录下创建pages目录，在里加新增一些.md后缀的文件。比如page-one.md:

```
This is a page.
```

我们需要写代码将这些文件的数据读取并展示出来。也许我们可以直接将代码都写在page控制器里，但要记住隔离关注原则，其他地方也可能读取这些文件。

所以我们将处理逻辑放在单独的文件中。还要考虑到，可能以后会将文件数据放在数据库中，所以我们可以先定义接口，具体的操作再基于这个接口来实现。

在src目录中新建Page目录，把相关的类放在这个目录下，先创建PageRender.php:

```php
<?php

namespace Example\Page;

interface PageReader
{
    public function readBySlug($slug);
}
```

文件读取的类实现，可以是FilePageReader.php:

```php
<?php

namespace Example\Page;

use InvalidArgumentException;

class FilePageReader implements PageReader
{
    private $pageFolder;

    public function __construct($pageFolder)
    {
        if (!is_string($pageFolder)) {
            throw new InvalidArgumentException('pageFolder must be a string');
        }
        $this->pageFolder = $pageFolder;
    }

    public function readBySlug($slug)
    {
        return 'I am a placeholder';
    }
}
```

你会发现我们将页面目录作为这个类的构造函数参数，这样当我们想改变页面存放目录或做单元测试的时候，就可以很方便的指定不同的参数了。

你也可以把Page相关的处理代码放在独立的包中，方便不同的应用程序重用。因为我们的目标就是松耦合。

因为php是弱类型语言，无法为变量提供类型提示功能，我们需要手动检查$pageFolder变量是否是字符串，如果不做检查，这很可能出现依赖注入相关的错误，检查可以抛出异常方便开发和调试。

现在我们在templates目录下创建Page.html,就加上{{content}}:

```php
{{content}}
```

然后就下面的代码加到Dependencies.php中，这样基于PageReader接口的实现类就可以被注入到处理逻辑中.

```php
$injector->define('Example\Page\FilePageReader', [
    ':pageFolder' => __DIR__ . '/../pages',
]);

$injector->alias('Example\Page\PageReader', 'Example\Page\FilePageReader');
$injector->share('Example\Page\FilePageReader');
```

修改Page控制器的show方法：

```php
public function show($params)
{
    $slug = $params['slug'];
    $data['content'] = $this->pageReader->readBySlug($slug);
    $html = $this->renderer->render('Page', $data);
    $this->response->setContent($html);
}
```

这里我们将Response，Renderer， PageReader 注入到Page控制器中。

```php
<?php

namespace Example\Controllers;

use Http\Response;
use Example\Template\Renderer;
use Example\Page\PageReader;

class Page
{
    private $response;
    private $renderer;
    private $pageReader;

    public function __construct(
        Response $response,
        Renderer $renderer,
        PageReader $pageReader
    ) {
        $this->response = $response;
        $this->renderer = $renderer;
        $this->pageReader = $pageReader;
    }
...
```

打开浏览器看看结果。FilePageReader已经生效,我们再来处理读取md部分。

检查readBySlug参数，必须是合法的字符串：

```php
public function readBySlug($slug)
{
    if (!is_string($slug)) {
        throw new InvalidArgumentException('slug must be a string');
    }
}
```

还需要加上页面无法找到的错误处理，在/src/Page目录下创建InvalidPageException.php：

```php
<?php

namespace Example\Page;

use Exception;

class InvalidPageException extends Exception
{
    public function __construct($slug, $code = 0, Exception $previous = null)
    {
        $message = "No page with the slug `$slug` was found";
        parent::__construct($message, $code, $previous);
    }
}
```

然后把判断加在readBySlug方法中：

```php
$path = "$this->pageFolder/$slug.md";

if(!file_exists($path)) {
    throw new InvalidPageException($slug);
}

return file_get_contents($path);
```

现在如果输入不存在的页面，会看到InvalidPageException异常。如果文件存在，就会看到内容。

当然把错误页面的异常信息抛给用户没有实际意义，我们就在捕获到异常时显示404页面。

```php
public function show($params)
{
    $slug = $params['slug'];

    try {
        $data['content'] = $this->pageReader->readBySlug($slug);
    } catch (InvalidPageException $e) {
        $this->response->setStatusCode(404);
        return $this->response->setContent('404 - Page not found');
    }
    
    $html = $this->renderer->render('Page', $data);
    $this->response->setContent($html);
}
```

将异常声明加上文件的顶部：
```php
use Example\Page\InvalidPageException;
```

这一点非常重要，否则会捕捉错误的异常。

打开浏览器，测试下是不是工作正常。多哆嗦一次，提交代码！

[<< 上一节](09-templating.md) | [下一节 >>](11-page-menu.md)
