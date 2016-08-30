[<< 上一节](11-page-menu.md) | [下一节 >>](to-be-continued.md)


### 前端

程序部分处理到上节已经差不多，是时候给网站改点样式了。

这里不讲前端的教程，大概使用一下[pure](http://purecss.io/).

先修改Layout.html

```php
<!doctype html>
<html lang="en">
    <head>
        <meta charset="utf-8">
        <title>Example</title>
        <link rel="stylesheet" href="http://yui.yahooapis.com/pure/0.6.0/pure-min.css">
        <link rel="stylesheet" href="/css/style.css">
    </head>
    <body>
        <div id="layout">
            <div id="menu">
                <div class="pure-menu">
                    <ul class="pure-menu-list">
                        {% for item in menuItems %}
                            <li class="pure-menu-item"><a href="{{ item['href'] }}" class="pure-menu-link">{{ item['text'] }}</a></li>
                        {% endfor %}
                    </ul>
                </div>
            </div>
            <div id="main">
                <div class="content">
                    {% block content %}
                    {% endblock %}
                </div>
            </div>
        </div>
    </body>
</html>
```

然后将自己的样式加在style.css文件里：

```css
body {
    color: #777;
}

#layout {
    position: relative;
    padding-left: 0;
}

#layout.active #menu {
    left: 150px;
    width: 150px;
}

#layout.active .menu-link {
    left: 150px;
}

.content {
    margin: 0 auto;
    padding: 0 2em;
    max-width: 800px;
    margin-bottom: 50px;
    line-height: 1.6em;
}

.header {
    margin: 0;
    color: #333;
    text-align: center;
    padding: 2.5em 2em 0;
    border-bottom: 1px solid #eee;
}

.header h1 {
    margin: 0.2em 0;
    font-size: 3em;
    font-weight: 300;
}

.header h2 {
    font-weight: 300;
    color: #ccc;
    padding: 0;
    margin-top: 0;
}

#menu {
    margin-left: -150px;
    width: 150px;
    position: fixed;
    top: 0;
    left: 0;
    bottom: 0;
    z-index: 1000; 
    background: #191818;
    overflow-y: auto;
    -webkit-overflow-scrolling: touch;
}

#menu a {
    color: #999;
    border: none;
    padding: 0.6em 0 0.6em 0.6em;
}

#menu .pure-menu,
#menu .pure-menu ul {
    border: none;
    background: transparent;
}

#menu .pure-menu ul,
#menu .pure-menu .menu-item-divided {
    border-top: 1px solid #333;
}

#menu .pure-menu li a:hover,
#menu .pure-menu li a:focus {
    background: #333;
}

#menu .pure-menu-selected,
#menu .pure-menu-heading {
    background: #1f8dd6;
}

#menu .pure-menu-selected a {
    color: #fff;
}

#menu .pure-menu-heading {
    font-size: 110%;
    color: #fff;
    margin: 0;
}

.header,
.content {
    padding-left: 2em;
    padding-right: 2em;
}

#layout {
    padding-left: 150px; /* left col width "#menu" */
    left: 0;
}
#menu {
    left: 150px;
}

.menu-link {
    position: fixed;
    left: 150px;
    display: none;
}

#layout.active .menu-link {
    left: 150px;
}
```

再看看网站，是不是感觉好多了。

所以静态相关的资源都可以放在public目录，比如可能会到的javascript,css,imges等等

（译者注：这里提到用户最好知道自己在哪个页面上，然后加页面，但后文没有具体展开）

现在可以加多一些页面了，在pages目录中创建page-one.md文件，然后修改菜单:

```php
return [
    ['href' => '/', 'text' => 'Homepage'],
    ['href' => '/page-one', 'text' => 'Page One'],
];
```

[<< 上一页](11-page-menu.md) | [下一页 >>](to-be-continued.md)

