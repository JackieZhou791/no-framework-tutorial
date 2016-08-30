[<< 上一节](10-dynamic-pages.md) | [下一节 >>](12-frontend.md)

### 页面菜单

上节中我们已经可以创建很多动态页面了，但是终端用户是不知道这些页面的链接的。所以本节我们来创建导航菜单，为了重用这个菜单，单独创建一个文件，其它的页面只用调用它。

从实践的角度来看，最开始如果我们使用一个包含具体模板的扩展模板引擎，这时会省下很多力气来手动创建模板文件，像layout,tempalte之类。很遗憾Mustache本身不提供这些，Twig就是不错的选择，好吧，下面我们使用Twig来完成本节。

[Twig](http://twig.sensiolabs.org/).

自行安装http://twig.sensiolabs.org/

在src/Template目录下创建TwigRenderer.php：

```php 
<?php

namespace Example\Template;

use Twig_Environment;

class TwigRenderer implements Renderer
{
    private $renderer;

    public function __construct(Twig_Environment $renderer)
    {
        $this->renderer = $renderer;
    }

    public function render($template, $data = [])
    {
        return $this->renderer->render("$template.html", $data);
    }
}
```

render方法中，模板文件加上了.html后缀，这是因为默认情况下twig模板文件没有后缀。将.html后缀加在这里，就可以和mustache引擎一样使用了。

修改依赖：

```php
$injector->delegate('Twig_Environment', function() use ($injector) {
    $loader = new Twig_Loader_Filesystem(dirname(__DIR__) . '/templates');
    $twig = new Twig_Environment($loader);
    return $twig;
});
```

这里我们使用delegate来创建类而不是define，这样的目的是可以在(匿名)函数中创建类.

然后修改Dependencies.php中Renderer别名

```php
$injector->alias('Example\Template\Renderer', 'Example\Template\TwigRenderer');

```

在Homepage控制器的show方法中修改$data值：

```php
$data = [
    'name' => $this->request->getParameter('name', 'stranger'),
    'menuItems' => [['href' => '/', 'text' => 'Homepage']],
];
```

将下面的代码放在Homepage.html文件的最上面：

```php
{% for item in menuItems %}
    <a href="{{ item.href }}">{{ item.text }}</a><br>
{% endfor %}
```

刷新页面，就可以看到链接了。

现在菜单只在首页有效，如果想在其他页面也有的话，复制到其他页面的模板中就可以了。不，这样做很不好。一旦菜单标签要修改，要修改所有的文件。

我们可以通过layout布局文件来处理这个,在templates目录下创建Layout.html:

```php
{% for item in menuItems %}
    <a href="{{ item['href'] }}">{{ item['text'] }}</a><br>
{% endfor %}
<hr>
{% block content %}
{% endblock %}
```

修改Homepage.html

```php
{% extends "Layout.html" %}
{% block content %}
    <h1>Hello World</h1>
    Hello {{ name }}
{% endblock %}
```

修改Page.html

```php
{% extends "Layout.html" %}
{% block content %}
    {{ content }}
{% endblock %}
```

刷新首页，菜单出现了。可是子页面没有，只是一个<hr>,这是因为menuItems只传给了首页，我们可以使用全局变量来处理，不过这样不太好。因为站点有很多不同的界面，前台/后台/管理端，这些都需要菜单，全局变量会变成麻烦事。

这时我们创建一个新的接口来处理不同的端的菜单需求：

```php
<?php

namespace Example\Template;

interface FrontendRenderer extends Renderer {}
```

基于新的接口的实现类，可以保证使用依赖注入时renderer的约束，也可以为新的render提供很多的方法实现上的差异。

现在创建新的实现类：

```php
<?php

namespace Example\Template;

class FrontendTwigRenderer implements FrontendRenderer
{
    private $renderer;

    public function __construct(Renderer $renderer)
    {
        $this->renderer = $renderer;
    }

    public function render($template, $data = [])
    {
        $data = array_merge($data, [
            'menuItems' => [['href' => '/', 'text' => 'Homepage']],
        ]);
        return $this->renderer->render($template, $data);
    }
}
```

这个类里的renderer作为依赖被注入，然后在render方法中将menuItems赋值给模板

将这个类加入到依赖文件中：

```php 
$injector->alias('Example\Template\FrontendRenderer', 'Example\Template\FrontendTwigRenderer');
```

然后将控制中的Renderer注入改为FrontendRenderer,确保文件中use了正确的完整路径。

删掉Homepage控制器中menuItems的赋值：

```php
'menuItems' => [['href' => '/', 'text' => 'Homepage']],
```

做完这些，菜单应该出现在所有页面上了。但是这样的重构并没有涉及菜单本身，所以还需要对菜单进行重构。因为考虑菜单很可能是存在在数据库中的，有必要对菜单进行分离。

创建/src/Menu目录，并创建MenuReader接口文件MenuReader.php：

```php
<?php

namespace Example\Menu;

interface MenuReader
{
    public function readMenu();
}
```

然后做一个简单的实现：

```php
<?php

namespace Example\Menu;

class ArrayMenuReader implements MenuReader
{
    public function readMenu()
    {
        return [
            ['href' => '/', 'text' => 'Homepage'],
        ];
    }
}
```

暂时先用这种简单的实现方式，后面我们再来细化。修改Dependencies.php:

```php
$injector->alias('Example\Menu\MenuReader', 'Example\Menu\ArrayMenuReader');
$injector->share('Example\Menu\ArrayMenuReader');
```

然后修改FrontendTwigRenderer，加入依赖注入：

```php
<?php

namespace Example\Template;

use Example\Menu\MenuReader;

class FrontendTwigRenderer implements FrontendRenderer
{
    private $renderer;
    private $menuReader;

    public function __construct(Renderer $renderer, MenuReader $menuReader)
    {
        $this->renderer = $renderer;
        $this->menuReader = $menuReader;
    }

    public function render($template, $data = [])
    {
        $data = array_merge($data, [
            'menuItems' => $this->menuReader->readMenu(),
        ]);
        return $this->renderer->render($template, $data);
    }
}
```

这样就实现了Menu的重构，打开浏览器看看。

[<< 上一节](10-dynamic-pages.md) | [下一节 >>](12-frontend.md)
