# Service Providers

- [简介](#introduction)
- [Provider 基础实例](#basic-provider-example)
- [注册 Providers](#registering-providers)
- [延迟 Providers](#deferred-providers)

<a name="introduction"></a>
## 简介（Introduction）

Service providers are the central place of all Laravel application bootstrapping. Your own application, as well as all of Laravel's core services are bootstrapped via service providers.

Service providers 是所有 Laravel 应用初始化运行的核心。你自己的应用和 Laravel 的核心服务一样全部都要通过 service providers 进行初始化。

But, what do we mean by "bootstrapped"? In general, we mean **registering** things, including registering service container bindings, event listeners, filters, and even routes. Service providers are the central place to configure your application.

我们这里所说的“初始化”又表示什么意思？大体来说，我们的意思是**注册**，包括对 service container 绑定的注册，事件监听器（event listeners），过滤器（filters），以及路由规则（routes）。Service Providers 是对你的应用进行配置的核心。


If you open the `config/app.php` file included with Laravel, you will see a `providers` array. These are all of the service provider classes that will be loaded for your application. Of course, many of them are "deferred" providers, meaning they will not be loaded on every request, but only when the services they provide are actually needed.

如果你打开 Laravel 中的 `config/app.php` 这个文件，你可以找到 `providers` 这个数组。这个数组中包含所有你的应用将要加载的 service provider 类。当然，其中某些属于“延迟使用”的，也就是说他们不会在每次请求时都被加载，只有当需要他们所提供的服务时才被加载。

In this overview you will learn how to write your own service providers and register them with your Laravel application.

在这篇概览中你将会学习如何在你的 Laravel 应用中创建并注册你自己的 service providers。

<a name="basic-provider-example"></a>
## Provider 基础实例（Basic Provider Example）

All service providers extend the `Illuminate\Support\ServiceProvider` class. This abstract class requires that you define at least one method on your provider: `register`. Within the `register` method, you should **only bind things into the [service container](/docs/5.0/container)**. You should never attempt to register any event listeners, routes, or any other piece of functionality within the `register` method.

所有的 service prvoders 都要继承 `Illuminate\Support\ServiceProvider` 类。这个抽象类要求必须定义一个 `register` 方法。 在 `register` 方法中，**只能将东西绑定到 [service container](/docs/5.0/container)**。不要尝试注册任何事件监听器，路由规则，以及任何功能性的代码。

The Artisan CLI can easily generate a new provider via the `make:provider` command:

使用 Artisan 命令行的 `make:provider` 命令可以很方便地生成一个新的 provider：

	php artisan make:provider RiakServiceProvider

### Register 方法（The Register Method）

Now, let's take a look at a basic service provider:

现在一起来看一段最基础的 service provider 代码：

	<?php namespace App\Providers;

	use Riak\Connection;
	use Illuminate\Support\ServiceProvider;

	class RiakServiceProvider extends ServiceProvider {

		/**
		 * Register bindings in the container.
		 *
		 * @return void
		 */
		public function register()
		{
			$this->app->singleton('Riak\Contracts\Connection', function($app)
			{
				return new Connection($app['config']['riak']);
			});
		}

	}

This service provider only defines a `register` method, and uses that method to define an implementation of `Riak\Contracts\Connection` in the service container. If you don't understand how the service container works, don't worry, [we'll cover that soon](/docs/5.0/container).

这个 service provider 只定义了 `register` 方法，并使用这个方法在 service container 中定义了一个 `Riak\Contracts\Connection` 的实例。如果你对容器的工作机制还不太清楚，不要担心，[我们很快就会介绍到它](/docs/5.0/container)。

This class is namespaced under `App\Providers` since that is the default location for service providers in Laravel. However, you are free to change this as you wish. Your service providers may be placed anywhere that Composer can autoload them.

这个类被定义在 Laravel 默认存放 service providers 的命名空间 `App\Providers` 中。但是也可以对它进行变更。你的 service providers 只可以被放置在任何 Composer 可以自动加载它们的地方。

### Boot 方法（The Boot Method）

So, what if we need to register an event listener within our service provider? This should be done within the `boot` method. **This method is called after all other service providers have been registered**, meaning you have access to all other services that have been registered by the framework.

如果我们需要在我们的 service provider 中注册事件监听器（event listener）怎么办？可以在 `boot` 方法中实现。**这个方法会在所有其他 service provider 完成注册后被调用**，也就是说在这个方法中可以访问所有其他已经被框架注册过的服务（services）。

	<?php namespace App\Providers;

	use Event;
	use Illuminate\Support\ServiceProvider;

	class EventServiceProvider extends ServiceProvider {

		/**
		 * Perform post-registration booting of services.
		 *
		 * @return void
		 */
		public function boot()
		{
			Event::listen('SomeEvent', 'SomeEventHandler');
		}

		/**
		 * Register bindings in the container.
		 *
		 * @return void
		 */
		public function register()
		{
			//
		}

	}

We are able to type-hint dependencies for our `boot` method. The service container will automatically inject any dependencies you need:

我们可以对 `boot` 方法的参数设置类型约束（type-hint）来定义依赖关系。service container 将会对所需要的依赖自动注入。

	use Illuminate\Contracts\Events\Dispatcher;

	public function boot(Dispatcher $events)
	{
		$events->listen('SomeEvent', 'SomeEventHandler');
	}

<a name="registering-providers"></a>
## 注册 Provider（Registering Providers）

All service providers are registered in the `config/app.php` configuration file. This file contains a `providers` array where you can list the names of your service providers. By default, a set of Laravel core service providers are listed in this array. These providers bootstrap the core Laravel components, such as the mailer, queue, cache, and others.

所有的 service providers 需要在 `config/app.php` 配置文件中注册。这个文件包含一个 `providers` 数组，可以在这里列出所有的 service providers。这个数组中默认列出了 Laravel 框架的核心 service providers。这些 providers 构成了 Laravel 的核心组件，例如 mailer（邮件），queue（队列），cache（缓存）等等。

To register your provider, simply add it to the array:

注册一个 provider 只需要把它添加到 `providers` 数组中即可：

	'providers' => [
		// Other Service Providers

		'App\Providers\AppServiceProvider',
	],

<a name="deferred-providers"></a>
## Providers 延迟（Deferred Providers）

If your provider is **only** registering bindings in the [service container](/docs/5.0/container), you may choose to defer its registration until one of the registered bindings is actually needed. Deferring the loading of such a provider will improve the performance of your application, since it is not loaded from the filesystem on every request.

如果一个 provider 仅用来在 [service container](/docs/5.0/container) 中注册绑定，你可以将它的注册做延迟处理，只在当有已注册的绑定真的需要它时才进行注册。延后这种 provider 的加载有利于提升应用的性能，因为它不会在每次请求时都从文件系统中被加载。

To defer the loading of a provider, set the `defer` property to `true` and define a `provides` method. The `provides` method returns the service container bindings that the provider registers:

给 provider 设置延迟加载的方法是，给 provider 的 `defer` 属性设置为 `true` 并定义一个 `provides` 方法。`provides` 方法返回这个 provider 注册的 service container 绑定。

	<?php namespace App\Providers;

	use Riak\Connection;
	use Illuminate\Support\ServiceProvider;

	class RiakServiceProvider extends ServiceProvider {

		/**
		 * Indicates if loading of the provider is deferred.
		 *
		 * @var bool
		 */
		protected $defer = true;

		/**
		 * Register the service provider.
		 *
		 * @return void
		 */
		public function register()
		{
			$this->app->singleton('Riak\Contracts\Connection', function($app)
			{
				return new Connection($app['config']['riak']);
			});
		}

		/**
		 * Get the services provided by the provider.
		 *
		 * @return array
		 */
		public function provides()
		{
			return ['Riak\Contracts\Connection'];
		}

	}

Laravel compiles and stores a list of all of the services supplied by deferred service providers, along with the name of its service provider class. Then, only when you attempt to resolve one of these services does Laravel load the service provider.

Laravel 会将所有延迟 service providers 提供的服务，编译并存储与一个列表中，以这些 service provider 的类名作为名字进行存储。那么，只有当你尝试解决其中一个服务时 Laravel 加载这个 service provider。

