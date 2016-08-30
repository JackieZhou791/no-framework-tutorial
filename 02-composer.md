[<< 上一节](01-front-controller.md) | [下一节 >>](03-error-handler.md)

### Composer

[Composer](https://getcomposer.org/)是一个php依赖管理器.

不使用框架进行开发，不是说我们要重新发明轮子。我们可以使用Composer下载第三方开发包，为我们的应用程序所有。

如果还没有装好Composer，请前往https://getcomposer.org/ 进行安装。安装好后，就可以到[Packagist](https://packagist.org/)网站找你需要的开发包了。

先创建一个空白的composer.json文件，放在网站的根目录下。composer.json必须是一个格式完整正确的json文件，它负责管理项目里的各种包依赖。

将下面的文件复制到composer.json中：

```json
{
    "name": "Project name",
    "description": "Your project description",
    "keywords": ["Your keyword", "Another keyword"],
    "license": "MIT",
    "authors": [
        {
            "name": "Your Name",
            "email": "your@email.com",
            "role": "Creator / Main Developer"
        }
    ],
    "require": {
        "php": ">=5.5.0"
    },
    "autoload": {
        "psr-4": {
            "Example\\": "src/"
        }
    }
}
```

上面的配置中使用了Example命名空间，你可以改为你自己想用的。

打开命令行控制台，进行网站根目录，输入composer update. 这个操作为产生一个composer.lock的文件，它会锁定项目依赖和当前vendor目录。

最好将composer.lock文件也提供到版本控制中，它可以帮忙某些集成测试工具，比如Travis CI进行同一版本的单元测试。它还可以确保同一项目的开发者使用同样版本的开发包。避免出现“在我的电脑上不一样”的问题。

现在基本搭好了这个空的项目，可以进行下一步的开发了。

[<< 上一节](01-front-controller.md) | [下一节 >>](03-error-handler.md)
