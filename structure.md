# 应用结构（Application Structure）

- [简介（Introduction）](#introduction)
- [根目录（The Root Directory）](#the-root-directory)
- [App 目录（The App Directory）](#the-app-directory)
- [设置命名空间（Namespacing Your Application）](#namespacing-your-application)

<a name="introduction"></a>
## 简介（Introduction）

Laravel 默认的应用结构，是试图提供一个对大型或小型项目都适用的基础。当然，你也可以根据自己的意愿去规划自己应用的代码结构。Laravel 几乎不对任何的类文件存放在哪里有什么限制，只要 Composer 可以自动加载这些类就可以。

<a name="the-root-directory"></a>
## 根目录（The Root Directory）

一个全新安装的 Laravel 应用的根目录，包含如下这些目录：

如你所想 `app` 目录，包含了整个应用的核心代码。稍后我们会对这个目录进行更深入的探索。

`bootstrap` 目录包含了一些框架初始化程序以及自动加载的配置。

`config` 目录，和目录的名字一样，存放着你的应用中所有的配置文件。

`database` 目录用于存放数据库的 migration 和 seeds。

`public` 目录用于存放所有前端的控制器以及网页的素材（例如：图片， JavaScript， CSS，等等）。

`resources` 目录用于存放视图文件，前端代码的原始文件（LESS， SASS， CoffeeScript）以及“语言包”。

`storage` 目录用于存放编译后的 Blade 模板文件，sessions 文件，缓存文件，以及各种由框架动态生成的文件。

`tests` 目录用于存放自动化测试的程序。

`vendor` 目录用于存放你的 Composer 所依赖的程序。

<a name="the-app-directory"></a>
## App 目录（The App Directory）

你的应用的“业务核心”代码都存放于 `app` 目录中。默认情况下，这个目录处于 `App` 这个命名空间下，并且可以被 Composer 通过 [PSR-4 自动加载标准](http://www.php-fig.org/psr/psr-4/) 自动加载。**可以使用 Artisan 命令：`app:name` 修改这个命名空间**。

`app` 目录下又许多子目录，例如 `Console`， `Http`， and `Providers`。我们可以把 `Console` 和 `Http` 认为是一个能够直达应用核心的 API。HTTP 协议和 CLI 是能够和你的应用进行交互的两套装置，只是不包含业务本身的罗技。换句话说，他们其实就是向应用发出命令的两种方式。`Console` 目录中包含所有的 Artisan 命令，同样，`HTTP` 目录中包含了所有的控制器（controllers），过滤器（filters），以及请求（requests）。

`Commands` 目录中存放着应用的所有命令程序（commands）。命令程序是指除了那些在当前请求周期中需要同步执行的任务外，可以被放在列队中异步执行的任务。

`Events` 目录同你预料的一样，用于存放事件相关的类。当然，不是一定要用类来实现事件；但是，如果你选择使用它们，通过 Artisan 命令行创建的事件类就会被默认放入这个目录。

`Handlers` 目录用于存放命令（commands）和（events）的处理器类。处理器收到一个命令或事件，并在收到这个命令或事件被触发的响应时，开始执行。

`Services` 目录中有各种各样的“辅助”服务，帮助应用实现各种功能。例如，Laravel 自带的 `Registrar` 就是用来验证和创建新用户的服务。其他的可能有提供与外部 API 交互的服务，metrics systems，或者是提供从你自己的应用中收集数据的服务。

`Exceptions` 目录用于存放进行异常处理的代码，同时也是一个捕获程序抛出的各种异常的好地方。

> **注意：** `app` 目录中的大多数类文件，都可以通过 Artisan 命令来生成。想了解这些命令，请在你自己的终端中执行 `php artisan list make` 命令。

<a name="namespacing-your-application"></a>
## 设置命名空间（Namespacing Your Application）

就像前面提到的，应用默认使用的命名空间是 `App`；但是，你也可以非常容易地通过运行 `app:name` 这个 Artisan 命令，来改变这个命名空间，使它与你的应用更匹配。例如，如果你的应用的名字是“SocialNet”，你就可以运行下面的命令：

	php artisan app:name SocialNet
