[下一节 >>](02-composer.md)

### 前端控制器

所谓 [前端控制器](http://en.wikipedia.org/wiki/Front_Controller_pattern), 就是应用程序的入口.

创建了应用程序目录，就需要应用程序入口去处理所有的请求，也就是需要一个index.php。

通常的办法是在网站要目录创建index.php文件，部分框架会这样处理，但是笔者并不推荐这种方式。

使用根目录的index.php作为程序入口，意味着他会放在Web服务器目录内，这样的Web服务器可以访问根目录下所有的子目录。当然如果加上合适的配置，也可以做个禁止HTTP请求访问某些存放代码的子目录。

然后有时候如果处理失误，整个应用程序代码就是直接暴露在HTTP讲求之下。

这时候如果在根目录中创建一个叫做public的子目录，同时创建src目录存放代码可能会更好。然后在public目录中创建index.php，只用在文件中引用下面的代码就可以避免应用程序代码的暴露

```php
<?php 

require __DIR__ . '/../src/Bootstrap.php';
```

__DIR__ [魔术变量](http://php.net/manual/en/language.constants.predefined.php) 指向当前文件目录，然后使用require来调用Bootstrap文件，这样就可以做到所有的文件调用都有同样的相对路径。相关如果在不同的目录访问index.php，就会出现调用位置的问题。

Bootstrap.php文件将应用程序中各个组件绑定在一起，简化访问。

public目录下可以用来存放应用程序所需的静态资源，比如javascript/css样式文件。

好了，现在前往src目录，创建Bootstrap.php文件

```php
<?php 

echo 'Hello World!';
```

好了，现在来看看这样的目录结构是否有问题。打开命令行控制台，进行入应用程序所在目录，进行public目录，输入php -S localhost:8000并回车，这时将会启动php内置的web服务器，然后我们通过浏览器访问http://localhost:8000。你应该已经看到’hello world’的输入了。

如果这里出现错误，仔细检查上面的操作步骤是不是有误。如果出现的是空白页，查看启动web服务器的命令行窗口，应该会出现相关的错误提示。

最后，可以提交下相关的代码了。推荐使用GIT来进行代码维护，如果你不太熟悉git,建议找一找git教程来学习下。版本控制应该成为一个开发习惯。

[下一节 >>](02-composer.md)
