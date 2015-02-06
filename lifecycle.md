# 请求的生命周期（Request Lifecycle）

- [简介（Introduction）](#introduction)
- [总览（Lifecycle Overview）](#lifecycle-overview)
- [关注于 Service Providers（Focus On Service Providers）](#focus-on-service-providers)

<a name="introduction"></a>
## 简介（Introduction）

在真实世界中无论使用任何工具，如果你对工具的工作原理有所了解就会在使用它们的时候更有把握。开发应用程序其实也一样。当你对所使用的开发工具的功能有所了解后，你在使用它们的时候会感到更自在和自信。

这篇文档的目的是让你从更高的角度对 Laravel 框架的工作原理有一个更好的理解。通过对整个框架更好的理解，使你不会对它再有一种处处充满“神秘”的感觉，也让你在使用它的时候更有信心。

如果你对这里所说的内容还不能全部理解，也不要灰心。试着先把握住最基础的东西，你对框架的了解会随着对文档其他部分的探索逐渐增加的。

<a name="lifecycle-overview"></a>
## 总览（Lifecycle Overview）

#### 第一步（First Things）

所有向 Laravel 应用发出的请求都会首先到达 `public/index.php` 这个入口文件。所有的请求都会被你的 web server (Apache / Nginx) 配置转向到这个文件。`index.php` 文件本身没有太多代码。相反，它仅仅作为一个起点来加载框架剩余的全部内容。

`index.php` 会加载又 Composer 生成的自动加载器，并且从 `bootstrap/app.php` 脚本中获取一个 Laravel application 的实例。Laravel 框架自己做的第一件事是创建一个应用 / [容器](/docs/5.0/container) 的实例。

#### HTTP / Console 核心（HTTP / Console Kernels）

接下来，请求被发送给 HTTP 或 console 核心，具体取决于这个请求的类型。这两个核心是所有请求流程必经的中心枢纽。这里，我们暂时只看 HTTP 核心，它位于 `app/Http/Kernel.php`。

HTTP 核心继承自 `Illuminate\Foundation\Http\Kernel` 类，这个类里定义了 `bootstrappers` 这个数组，它会在整个请求之前被之情。这些 bootstrappers 进行了错误处理的配置，日志配置，检测应用的运行环境，然后执行一些需要在请求真正被执行前需要处理的任务。

HTTP 核心同样也定义了一组所有请求在到达应用处理环节前必须经过的 HTTP [中间件](/docs/5.0/middleware)。这些中间件处理对 HTTP session 的读写，判断应用是否处于维护状态，验证 CSRF token，等等。

HTTP 核心中的 `handle` 方法的任务很简单：接收请求 `Request` 然后返回应答 `Response`。把核心想象成代表你整个应用的一个巨大的黑盒。给它输入 HTTP 请求，它将返回 HTTP 应答。

#### Service Providers

其中一个最重要的核心引导动作是为应用加载 service providers。所有的 service providers 都是在 `config/app.php` 配置文件中的 `providers` 数组里配置的。首先，所有 providers 的 `register` 方法会被调用，一旦所有 providers 都被注册完，就会调用 `boot` 方法。

#### Dispatch Request

一旦程序已经被引导，所有的 service providers 完成注册，请求 `Request` 就会被交给路由器（router）等待分派。路由器会把请求发送到一个路由规则或控制器，以及运行路由规则指定的中间件。

<a name="focus-on-service-providers"></a>
## Focus On Service Providers

Service providers 是一个 Laravel 应用引导的核心。创建应用的实例，注册 Service providers，将请求交给完成引导的应用。就是这么简单！

牢牢掌握 Laravel 应用如何通过 Service providers 被建立并完成引导是非常有价值的。对了，应用默认的 Service providers 都存放于 `app/Providers` 目录中。

`AppServiceProvider` 默认是没有内容的。这个 provider 是一个非常适合添加我们自己定义的引导项和容器绑定的地方。当然，对于大型应用来说，你也许希望创建一些 Service providers，每一个 Service provider 都有一个更加颗粒化的引导功能。
