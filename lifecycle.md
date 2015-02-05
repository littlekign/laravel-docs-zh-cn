# 请求的生命周期（Request Lifecycle）

- [简介（Introduction）](#introduction)
- [总览（Lifecycle Overview）](#lifecycle-overview)
- [关注于 Service Providers（Focus On Service Providers）](#focus-on-service-providers)

<a name="introduction"></a>
## 简介（Introduction）

When using any tool in the "real world", you feel more confident if you understand how that tool works. Application development is no different. When you understand how your development tools function, you feel more comfortable and confident using them.

在真实世界中无论使用任何工具，如果你对工具的工作原理有所了解就会在使用它们的时候更有把握。开发应用程序其实也一样。当你对所使用的开发工具的功能有所了解后，你在使用它们的时候会感到更自在和自信。

The goal of this document is to give you a good, high-level overview of how the Laravel framework "works". By getting to know the overall framework better, everything feels less "magical" and you will be more confident building your applications.

这篇文档的目的是让你从更高的角度对 Laravel 框架的工作原理有一个更好的理解。通过对整个框架更好的理解，使你不会对它再有一种处处充满“神秘”的感觉，也让你在使用它的时候更有信心。

If you don't understand all of the terms right away, don't lose heart! Just try to get a basic grasp of what is going on, and your knowledge will grow as you explore other sections of the documentation.

如果你对这里所说的内容还不能全部理解，也不要灰心。试着先把握住最基础的东西，你对框架的了解会随着对文档其他部分的探索逐渐增加的。

<a name="lifecycle-overview"></a>
## 总览（Lifecycle Overview）

#### 第一步（First Things）

The entry point for all requests to a Laravel application is the `public/index.php` file. All requests are directed to this file by your web server (Apache / Nginx) configuration. The `index.php` file doesn't contain much code. Rather, it is simply a starting point for loading the rest of the framework.

所有向 Laravel 应用发出的请求都会首先到达 `public/index.php` 这个入口文件。所有的请求都会被你的 web server (Apache / Nginx) 配置转向到这个文件。`index.php` 文件本身没有太多代码。相反，它仅仅作为一个起点来加载框架剩余的全部内容。

The `index.php` file loads the Composer generated autoloader definition, and then retrieves an instance of the Laravel application from `bootstrap/app.php` script. The first action taken by Laravel itself is to create an instance of the application / [service container](/docs/5.0/container).

`index.php` 会加载又 Composer 生成的自动加载器，并且从 `bootstrap/app.php` 脚本中获取一个 Laravel application 的实例。Laravel 框架自己做的第一件事是创建一个应用 / [容器](/docs/5.0/container) 的实例。

#### HTTP / Console 核心（HTTP / Console Kernels）

Next, the incoming request is sent to either the HTTP kernel or the console kernel, depending on the type of request that is entering the application. These two kernels serve as the central location that all requests flow through. For now, let's just focus on the HTTP kernel, which is located in `app/Http/Kernel.php`.

接下来，请求被发送给 HTTP 或 console 核心，具体取决于这个请求的类型。这两个核心是所有请求流程必经的中心枢纽。这里，我们暂时只看 HTTP 核心，它位于 `app/Http/Kernel.php`。

The HTTP kernel extends the `Illuminate\Foundation\Http\Kernel` class, which defines an array of `bootstrappers` that will be run before the request is executed. These bootstrappers configure error handling, configure logging, detect the application environment, and perform other tasks that need to be done before the request is actually handled.

HTTP 核心继承自 `Illuminate\Foundation\Http\Kernel` 类，这个类里定义了 `bootstrappers` 这个数组，它会在整个请求之前被之情。这些 bootstrappers 进行了错误处理的配置，日志配置，检测应用的运行环境，然后执行一些需要在请求真正被执行前需要处理的任务。

The HTTP kernel also defines a list of HTTP [middleware](/docs/5.0/middleware) that all requests must pass through before being handled by the application. These middleware handle reading and writing the HTTP session, determine if the application is in maintenance mode, verifying the CSRF token, and more.

HTTP 核心同样也定义了一组所有请求在到达应用处理环节前必须经过的 HTTP [中间件](/docs/5.0/middleware)。这些中间件处理对 HTTP session 的读写，判断应用是否处于维护状态，验证 CSRF token，等等。

The method signature for the HTTP kernel's `handle` method is quite simple: receive a `Request` and return a `Response`. Think of the Kernel as being a big black box that represents your entire application. Feed it HTTP requests and it will return HTTP responses.

HTTP 核心中的 `handle` 方法的任务很简单：接收请求 `Request` 然后返回应答 `Response`。把核心想象成代表你整个应用的一个巨大的黑盒。给它输入 HTTP 请求，它将返回 HTTP 应答。

#### Service Providers

One of the most important Kernel bootstrapping actions is loading the service providers for your application. All of the service providers for the application are configured in the `config/app.php` configuration file's `providers` array. First, the `register` method will be called on all providers, then, once all providers have been registered, the `boot` method will be called.

其中一个最重要的核心引导动作是为应用加载 service providers。所有的 service providers 都是在 `config/app.php` 配置文件中的 `providers` 数组里配置的。首先，所有 providers 的 `register` 方法会被调用，一旦所有 providers 都被注册完，就会调用 `boot` 方法。

#### Dispatch Request

Once the application has been bootstrapped and all service providers have been registered, the `Request` will be handed off to the router for dispatching. The router will dispatch the request to a route or controller, as well as run any route specific middleware.

一旦程序已经被引导，所有的 service providers 完成注册，请求 `Request` 就会被交给路由器（router）等待分派。路由器会把请求发送到一个路由规则或控制器，以及运行路由规则指定的中间件。

<a name="focus-on-service-providers"></a>
## Focus On Service Providers

Service providers are truly the key to bootstrapping a Laravel application. The application instance is created, the service providers are registered, and the request is handed to the bootstrapped application. It's really that simple!

Service providers 是一个 Laravel 应用引导的核心。创建应用的实例，注册 Service providers，将请求交给完成引导的应用。就是这么简单！

Having a firm grasp of how a Laravel application is built and bootstrapped via service providers is very valuable. Of course, your application's default service providers are stored in the `app/Providers` directory.

牢牢掌握 Laravel 应用如何通过 Service providers 被建立并完成引导是非常有价值的。对了，应用默认的 Service providers 都存放于 `app/Providers` 目录中。

By default, the `AppServiceProvider` is fairly empty. This provider is a great place to add your application's own bootstrapping and service container bindings. Of course, for large applications, you may wish to create several service providers, each with a more granular type of bootstrapping.

`AppServiceProvider` 默认是没有内容的。这个 provider 是一个非常适合添加我们自己定义的引导项和容器绑定的地方。当然，对于大型应用来说，你也许希望创建一些 Service providers，每一个 Service provider 都有一个更加颗粒化的引导功能。