# 合约（Contracts）

- [简介（Introduction）](#introduction)
- [为何使用合约？（Why Contracts?）](#why-contracts)
- [合约索引（Contract Reference）](#contract-reference)
- [如何使用合约（How To Use Contracts）](#how-to-use-contracts)

<a name="introduction"></a>
## 简介（Introduction）

Laravel's Contracts are a set of interfaces that define the core services provided by the framework. For example, a `Queue` contract defines the methods needed for queueing jobs, while the `Mailer` contract defines the methods needed for sending e-mail.

Laravel 的合约是一组用来定义框架核心服务的接口。例如，`Queue` 的合约定义了队列任务所需的方法，而 `Mailer` 的合约则定义了邮件发送需要的方法。

Each contract has a corresponding implementation provided by the framework. For example, Laravel provides a `Queue` implementation with a variety of drivers, and a `Mailer` implementation that is powered by [SwiftMailer](http://swiftmailer.org/).

对应每一个合约都有一个框架提供的实现。例如，Laravel 提供了一个包含多种驱动的 `Queue` 的实例，而 `Mailer` 的实例则是由 [SwiftMailer](http://swiftmailer.org/) 驱动的。

All of the Laravel contracts live in [their own GitHub repository](https://github.com/illuminate/contracts). This provides a quick reference point for all available contracts, as well as a single, decoupled package that may be utilized by other package developers.

Laravel 的所有合约都存放在[他们自己的 GitHub 仓库中](https://github.com/illuminate/contracts)。作为一个独立的，由许多其他开发者贡献的程序包共同组成的框架，这里提供一个可以快速到达所有可以使用的合约的索引列表，

<a name="why-contracts"></a>
## 为何使用合约？（Why Contracts?）

You may have several questions regarding contracts. Why use interfaces at all? Isn't using interfaces more complicated?

对于合约你可能会有一些问题。为什么要全部使用接口？用接口不是会更复杂吗？

Let's distill the reasons for using interfaces to the following headings: loose coupling and simplicity.

使用接口的原因可以提炼为：解藕和简化。

### 解藕（Loose Coupling）

First, let's review some code that is tightly coupled to a cache implementation. Consider the following:

先来看一段与缓存实例有强耦合关系的代码，如下：

	<?php namespace App\Orders;

	class Repository {

		/**
		 * The cache.
		 */
		protected $cache;

		/**
		 * Create a new repository instance.
		 *
		 * @param  \Package\Cache\Memcached  $cache
		 * @return void
		 */
		public function __construct(\SomePackage\Cache\Memcached $cache)
		{
			$this->cache = $cache;
		}

		/**
		 * Retrieve an Order by ID.
		 *
		 * @param  int  $id
		 * @return Order
		 */
		public function find($id)
		{
			if ($this->cache->has($id))
			{
				//
			}
		}

	}

In this class, the code is tightly coupled to a given cache implementation. It is tightly coupled because we are depending on a concrete Cache class from a package vendor. If the API of that package changes our code must change as well.

在这个类里，代码与一个指定的缓存实例绑定。因为我们依赖于一个实体的第三方程序包的缓存类，而造成这种强耦合的关系。如果这个第三方程序的接口发生变化我们的代码也要进行相应的调整。

Likewise, if we want to replace our underlying cache technology (Memcached) with another technology (Redis), we again will have to modify our repository. Our repository should not have so much knowledge regarding who is providing them data or how they are providing it.

同样的，如果我们想要用另外一种缓存方案（Redis）替换正在使用的缓存方案（Memcached），就需要再次修改我们的代码。但是我们的代码其实并不需要具体了解谁提供数据或是它们如何提供数据。

**Instead of this approach, we can improve our code by depending on a simple, vendor agnostic interface:**

**取而代之，我们可以通过依靠一个简单的，与具体提供者无关的接口来改进代码：**

	<?php namespace App\Orders;

	use Illuminate\Contracts\Cache\Repository as Cache;

	class Repository {

		/**
		 * Create a new repository instance.
		 *
		 * @param  Cache  $cache
		 * @return void
		 */
		public function __construct(Cache $cache)
		{
			$this->cache = $cache;
		}

	}

Now the code is not coupled to any specific vendor, or even Laravel. Since the contracts package contains no implementation and no dependencies, you may easily write an alternative implementation of any given contract, allowing you to replace your cache implementation without modifying any of your cache consuming code.

现在的代码与任何实际的提供者，甚至 Laravel 框架都无关了。因为合约包中不包含任何实例以及其他依赖，你可以轻松的为一个合约写出一个可选的实现，并允许你在不修改任何使用缓存服务代码的情况下替换成你自己实现的缓存实例。

### 简化（Simplicity）

When all of Laravel's services are neatly defined within simple interfaces, it is very easy to determine the functionality offered by a given service. **The contracts serve as succinct documentation to the framework's features.**

当 Laravel 的所有服务都在接口中进行简明的定义后，就很容易判断一个给定服务的功能了。**而合约只作为框架的功能在文档中进行简洁的记录。**

In addition, when you depend on simple interfaces, your code is easier to understand and maintain. Rather than tracking down which methods are available to you within a large, complicated class, you can refer to a simple, clean interface.

此外，当使用一个简单的接口时，你的代码就会更容易理解和维护。相较以前需要在一个既大又复杂的类里面去追踪到底哪些方法是你可以使用的，现在你只需要去查看一个简单，清晰的接口。

<a name="contract-reference"></a>
## 合约索引（Contract Reference）

This is a reference to most Laravel Contracts, as well as their Laravel "facade" counterparts:

这是一份对大部分的 Laravel 合约的索引，以及它们与 Laravel 中“facade”的对应关系：

Contract  |  Laravel 4.x Facade
------------- | -------------
[Illuminate\Contracts\Auth\Guard](https://github.com/illuminate/contracts/blob/master/Auth/Guard.php)  |  Auth
[Illuminate\Contracts\Auth\PasswordBroker](https://github.com/illuminate/contracts/blob/master/Auth/PasswordBroker.php)  |  Password
[Illuminate\Contracts\Cache\Repository](https://github.com/illuminate/contracts/blob/master/Cache/Repository.php) | Cache
[Illuminate\Contracts\Cache\Factory](https://github.com/illuminate/contracts/blob/master/Cache/Factory.php) | Cache::driver()
[Illuminate\Contracts\Config\Repository](https://github.com/illuminate/contracts/blob/master/Config/Repository.php) | Config
[Illuminate\Contracts\Container\Container](https://github.com/illuminate/contracts/blob/master/Container/Container.php) | App
[Illuminate\Contracts\Cookie\Factory](https://github.com/illuminate/contracts/blob/master/Cookie/Factory.php) | Cookie
[Illuminate\Contracts\Cookie\QueueingFactory](https://github.com/illuminate/contracts/blob/master/Cookie/QueueingFactory.php) | Cookie::queue()
[Illuminate\Contracts\Encryption\Encrypter](https://github.com/illuminate/contracts/blob/master/Encryption/Encrypter.php) | Crypt
[Illuminate\Contracts\Events\Dispatcher](https://github.com/illuminate/contracts/blob/master/Events/Dispatcher.php) | Event
[Illuminate\Contracts\Filesystem\Cloud](https://github.com/illuminate/contracts/blob/master/Filesystem/Cloud.php) | &nbsp;
[Illuminate\Contracts\Filesystem\Factory](https://github.com/illuminate/contracts/blob/master/Filesystem/Factory.php) | File
[Illuminate\Contracts\Filesystem\Filesystem](https://github.com/illuminate/contracts/blob/master/Filesystem/Filesystem.php) | File
[Illuminate\Contracts\Foundation\Application](https://github.com/illuminate/contracts/blob/master/Foundation/Application.php) | App
[Illuminate\Contracts\Hashing\Hasher](https://github.com/illuminate/contracts/blob/master/Hashing/Hasher.php) | Hash
[Illuminate\Contracts\Logging\Log](https://github.com/illuminate/contracts/blob/master/Logging/Log.php) | Log
[Illuminate\Contracts\Mail\MailQueue](https://github.com/illuminate/contracts/blob/master/Mail/MailQueue.php) | Mail::queue()
[Illuminate\Contracts\Mail\Mailer](https://github.com/illuminate/contracts/blob/master/Mail/Mailer.php) | Mail
[Illuminate\Contracts\Queue\Factory](https://github.com/illuminate/contracts/blob/master/Queue/Factory.php) | Queue::driver()
[Illuminate\Contracts\Queue\Queue](https://github.com/illuminate/contracts/blob/master/Queue/Queue.php) | Queue
[Illuminate\Contracts\Redis\Database](https://github.com/illuminate/contracts/blob/master/Redis/Database.php) | Redis
[Illuminate\Contracts\Routing\Registrar](https://github.com/illuminate/contracts/blob/master/Routing/Registrar.php) | Route
[Illuminate\Contracts\Routing\ResponseFactory](https://github.com/illuminate/contracts/blob/master/Routing/ResponseFactory.php) | Response
[Illuminate\Contracts\Routing\UrlGenerator](https://github.com/illuminate/contracts/blob/master/Routing/UrlGenerator.php) | URL
[Illuminate\Contracts\Support\Arrayable](https://github.com/illuminate/contracts/blob/master/Support/Arrayable.php) | &nbsp;
[Illuminate\Contracts\Support\Jsonable](https://github.com/illuminate/contracts/blob/master/Support/Jsonable.php) | &nbsp;
[Illuminate\Contracts\Support\Renderable](https://github.com/illuminate/contracts/blob/master/Support/Renderable.php) | &nbsp;
[Illuminate\Contracts\Validation\Factory](https://github.com/illuminate/contracts/blob/master/Validation/Factory.php) | Validator::make()
[Illuminate\Contracts\Validation\Validator](https://github.com/illuminate/contracts/blob/master/Validation/Validator.php) | &nbsp;
[Illuminate\Contracts\View\Factory](https://github.com/illuminate/contracts/blob/master/View/Factory.php) | View::make()
[Illuminate\Contracts\View\View](https://github.com/illuminate/contracts/blob/master/View/View.php) | &nbsp;

<a name="how-to-use-contracts"></a>
## 如何使用合约（How To Use Contracts）

So, how do you get an implementation of a contract? It's actually quite simple. Many types of classes in Laravel are resolved through the [service container](/docs/master/container), including controllers, event listeners, filters, queue jobs, and even route Closures. So, to get an implementation of a contract, you can just "type-hint" the interface in the constructor of the class being resolved. For example, take a look at this event handler:

那么，如何获得一个合约的实例？其实很简单。在 Laravel 中很多类型的类都通过[容器](/docs/master/container)调用，包括 ontrollers, event listeners, filters, queue jobs, 以及 route Closures。所以，你可以直接在被调用类的构造方法中用“类型约定”的方式绑定接口，以此来获得一个合约的实例。例如，下面这个事件处理器的例子：

	<?php namespace App\Handlers\Events;

	use App\User;
	use App\Events\NewUserRegistered;
	use Illuminate\Contracts\Redis\Database;

	class CacheUserInformation {

		/**
		 * The Redis database implementation.
		 */
		protected $redis;

		/**
		 * Create a new event handler instance.
		 *
		 * @param  Database  $redis
		 * @return void
		 */
		public function __construct(Database $redis)
		{
			$this->redis = $redis;
		}

		/**
		 * Handle the event.
		 *
		 * @param  NewUserRegistered  $event
		 * @return void
		 */
		public function handle(NewUserRegistered $event)
		{
			//
		}

	}

When the event listener is resolved, the service container will read the type-hints on the constructor of the class, and inject the appropriate value. To learn more about registering things in the service container, check out [the documentation](/docs/master/container).

当事件监听器被调用时，容器将会从类的构造方法中读取“类型约定”的内容，并注入相应的值。了解更多关于在容器中注册的信息，请查看[容器相关文档](/docs/master/container)
