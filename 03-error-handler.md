[<< 上一节](02-composer.md) | [下一节 >>](04-http.md)

### 错误处理

代码中的定制错误处理，可以帮助我们快速定位各种类型的错误。

好的错误处理最好可以帮忙代码的调试，所以第一个项目一定要注意它的处理。

笔者喜欢 [filp/whoops](https://github.com/filp/whoops), 本系统教程里会使用它。如果你中意别的开发包，也可以选择使用其他的，这就是定制开发的好处，完全控制自己想要的代码。

另外也可以尝试 [PHP-Error](https://github.com/JosephLenton/PHP-Error)

在composer.json将的require block中增加所要的包信息，进行安装
```php
"require": {
    "php": ">=5.5.0",
    "filp/whoops": ">=1.1.2"
},
```

在命令行中输入composer.update， filp/whoops包就会被下载安装

但是还不能使用它，PHP不知道如何找到对应的文件。这时候我们需要一个autoloader自动加载器，最理想的是 [PSR-4](http://www.php-fig.org/psr/psr-4/) 加载器. Composer已经为我们处理好，我们只需要将下面的代码加入到Bootstrap.php文件中：
```php
require __DIR__ . '/../vendor/autoload.php’;
```

**重要提示:** 不要在生产环境中显示任何错误信息，一串错误追溯信息甚至一条简单的错误提示都有可能让别人获得你的系统权限。使用友好的错误提示或发送一封邮件到管理员邮箱，或者写日志等方式，确保只有网站管理员有权限看到生产环境的错误信息。

开发环境则不同，需要大量的错误提示信息。增加environment变量，在代码中进行切换的办法可以方便的实现分享，现在我们将environment设置为development。

注册错误处理类之后，可以抛出一个异常来测试是否工作正常，现在Bootstrap.php文件应该是这样：

```php
<?php

namespace Example;

require __DIR__ . '/../vendor/autoload.php';

error_reporting(E_ALL);

$environment = 'development';

/**
* Register the error handler
*/
$whoops = new \Whoops\Run;
if ($environment !== 'production') {
    $whoops->pushHandler(new \Whoops\Handler\PrettyPageHandler);
} else {
    $whoops->pushHandler(function($e){
        echo 'Friendly error page and send an email to the developer';
    });
}
$whoops->register();

throw new \Exception;

```

你应该已经看到一个有高度提示的错误页面。如果没有，请检查上面的步骤。

[<< 上一节](02-composer.md) | [下一节 >>](04-http.md)
