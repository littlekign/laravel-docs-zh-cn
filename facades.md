# Facades（外观）

- [简介（Introduction）](#introduction)
- [详解（Explanation）](#explanation)
- [实例（Practical Usage）](#practical-usage)
- [创建 Facades（Creating Facades）](#creating-facades)
- [模拟 Facades（Mocking Facades）](#mocking-facades)
- [Facade 类索引（Facade Class Reference）](#facade-class-reference)

<a name="introduction"></a>
## 简介（Introduction）

Facades provide a "static" interface to classes that are available in the application's [IoC container](/docs/5.0/container). Laravel ships with many facades, and you have probably been using them without even knowing it! Laravel "facades" serve as "static proxies" to underlying classes in the IoC container, providing the benefit of a terse, expressive syntax while maintaining more testability and flexibility than traditional static methods.

Facades 给那些在应用的 [IoC 容器](/doc/5.0/container)中可用的类提供一个静态接口。Laravel 自带了很多 Facades，而且你很可能已经在不知不觉中用到了它们！在 Laravel 中 ，Facades 为那些在 Ioc 容器中的基础类提供一种类似于“静态代理”的功能，它能够提供一种更简洁的，富于表达的语法，使维护的代码相较传统静态方法更灵活，更易于测试。

Occasionally, You may wish to create your own facades for your applications and packages, so let's explore the concept, development and usage of these classes.

偶尔，你可能希望为自己应的用或程序包创建 Facade，下面我们就一起看一下 Facade 的实现思路，如何开发以及使用它。

> **Note:** Before digging into facades, it is strongly recommended that you become very familiar with the Laravel [IoC container](/docs/5.0/container).

> **注意：** 在开始更深入研究 facades 之前，强烈建议你先对 Laravel 的[IoC 容器](/docs/5.0/container)有一个比较全面的了解。

<a name="explanation"></a>
## 详解（Explanation）

In the context of a Laravel application, a facade is a class that provides access to an object from the container. The machinery that makes this work is in the `Facade` class. Laravel's facades, and any custom facades you create, will extend the base `Facade` class.

在一个 Laravel 应用中，一个 facade 就是一个能通过容器来访问某一个对象的类。这种工作机制实际是由 `Facade` 这个类实现的，Laravel 本身的 facades 以及任何你自己创建的 facades 都需要继承 `Facade` 这个类。

Your facade class only needs to implement a single method: `getFacadeAccessor`. It's the `getFacadeAccessor` method's job to define what to resolve from the container. The `Facade` base class makes use of the `__callStatic()` magic-method to defer calls from your facade to the resolved object.

我们自己编写的 facade 类只需要实现 `getFacadeAccessor` 这个方法即可。`getFacadeAccessor` 主要是用来定义从容器中获取什么。而 `Facade` 类则使用 `__callStatic()` 方法将我们的 facade 转换成最终需要调用的对象。

So, when you make a facade call like `Cache::get`, Laravel resolves the Cache manager class out of the IoC container and calls the `get` method on the class. In technical terms, Laravel Facades are a convenient syntax for using the Laravel IoC container as a service locator.

所以，例如我们编写了一个 `Cache::get` 的 facade，Laravel 就会从 IoC 容器中获得缓存管理的类，并调用这个类中的 `get` 方法。从技术的角度来看，Laravel Facades 其实是当我们把 IoC 容器当作一个服务定位器使用时，一种简便的语法形式。

<a name="practical-usage"></a>
## 实例（Practical Usage）

In the example below, a call is made to the Laravel cache system. By glancing at this code, one might assume that the static method `get` is being called on the `Cache` class.

在下面的例子中，发起了一个对 Laravel 缓存系统的调用。粗看这段代码你可能会认为它是调用了 `Cache` 类的 `get` 这个静态方法。

	$value = Cache::get('key');

However, if we look at that `Illuminate\Support\Facades\Cache` class, you'll see that there is no static method `get`:

但是，如果我们去看一下 `Illuminate\Support\Facades\Cache` 类，你会发现这个类里没有 `get` 这个静态方法：

	class Cache extends Facade {

		/**
		 * Get the registered name of the component.
		 *
		 * @return string
		 */
		protected static function getFacadeAccessor() { return 'cache'; }

	}

The Cache class extends the base `Facade` class and defines a method `getFacadeAccessor()`. Remember, this method's job is to return the name of an IoC binding.

Cache 这个类继承自 `Facade` 这个基类，并且定义了 `getFacadeAccessor()` 方法。记住，这个方法的工作是返回一个 IoC 绑定的名字。

When a user references any static method on the `Cache` facade, Laravel resolves the `cache` binding from the IoC container and runs the requested method (in this case, `get`) against that object.

当用户引用 `Cache` facade 的任何静态方法时，Laravel 都会从 IoC 容器中获取 `cache` 绑定，并执行这个对象中对应请求的方法（这个例子中是 `get`）。

So, our `Cache::get` call could be re-written like so:

所以，`Cache::get` 这样的调用方式也可以这样来实现：

	$value = $app->make('cache')->get('key');

#### 引入 Facades（Importing Facades）

Remember, if you are using a facade when a controller that is namespaced, you will need to import the facade class into the namespace. All facades live in the global namespace:

记住，如果正在使用一个处于某个命名空间中的控制器时，你需要先引入需要使用的 facade 类的命名空间。所有的 facades 在全局命名空间中都是可用的：

	<?php namespace App\Http\Controllers;

	use Cache;

	class PhotosController extends Controller {

		/**
		 * Get all of the application photos.
		 *
		 * @return Response
		 */
		public function index()
		{
			$photos = Cache::get('photos');

			//
		}

	}

<a name="creating-facades"></a>
## 创建 Facades（Creating Facades）

Creating a facade for your own application or package is simple. You only need 3 things:

为你自己的应用或程序包新建一个 facade 很容易。只需要三样东西：

- An IoC binding.
- 一个 IoC 绑定。

- A facade class.
- 一个 facade 类。

- A facade alias configuration.
- 一个 facade 的别名配置。

Let's look at an example. Here, we have a class defined as `PaymentGateway\Payment`.

一起来看个例子。首先，我们有一个定义为 `PaymentGateway\Payment` 的类。

	namespace PaymentGateway;

	class Payment {

		public function process()
		{
			//
		}

	}

We need to be able to resolve this class from the IoC container. So, let's add a binding to a service provider:

我们需要能够从 IoC 容器中取得这个类，所以，先要把它绑定到一个 service provider 上：

	App::bind('payment', function()
	{
		return new \PaymentGateway\Payment;
	});

A great place to register this binding would be to create a new [service provider](/docs/5.0/container#service-providers) named `PaymentServiceProvider`, and add this binding to the `register` method. You can then configure Laravel to load your service provider from the `config/app.php` configuration file.

一个注册这个绑定很好的方式是，新建一个名为 `PaymentServiceProvider` 的 [service provider](/docs/5.0/container#service-providers)，并将这个绑定添加到 `register` 方法中。然后就可以在 `config/app.php` 中进行配置，并让你的应用加载上这个 service provider 了。

Next, we can create our own facade class:

接下来，就可以创建我们自己的 facade 类了：

	use Illuminate\Support\Facades\Facade;

	class Payment extends Facade {

		protected static function getFacadeAccessor() { return 'payment'; }

	}

Finally, if we wish, we can add an alias for our facade to the `aliases` array in the `config/app.php` configuration file. Now, we can call the `process` method on an instance of the `Payment` class.

最后，如果愿意的话，还可以在配置文件 `config/app.php` 的 `aliases` 数组中给我们的 facade 添加一个别名。现在，我们就可以对 `Payment` 类的实例调用 `process` 方法了。

	Payment::process();

### 关于别名自动加载中，一点需要注意的问题（A Note On Auto-Loading Aliases）

Classes in the `aliases` array are not available in some instances because [PHP will not attempt to autoload undefined type-hinted classes](https://bugs.php.net/bug.php?id=39003). If `\ServiceWrapper\ApiTimeoutException` is aliased to `ApiTimeoutException`, a `catch(ApiTimeoutException $e)` outside of the namespace `\ServiceWrapper` will never catch the exception, even if one is thrown. A similar problem is found in classes which have type hints to aliased classes. The only workaround is to forego aliasing and `use` the classes you wish to type hint at the top of each file which requires them.

因为 [PHP will not attempt to autoload undefined type-hinted classes](https://bugs.php.net/bug.php?id=39003) 这个 bug，`aliases` 数组中的类在某些实例中是不可用的。如果 `\ServiceWrapper\ApiTimeoutException` 被定义了 `ApiTimeoutException` 别名，在 `\ServiceWrapper` 命名空间之外的 `catch(ApiTimeoutException $e)` 都无法正确捕获错误，即使错误已经被抛出。当一个类，它通过类型约定绑定到另一个有别名的类时，也会产生类似的问题。这种情况下唯一的方案是放弃使用别名，而是在每个需要使用他们的文件头部用 `use` 的方式来指定要用的类。

<a name="mocking-facades"></a>
## 模拟 Facades（Mocking Facades）

Unit testing is an important aspect of why facades work the way that they do. In fact, testability is the primary reason for facades to even exist. For more information, check out the [mocking facades](/docs/testing#mocking-facades) section of the documentation.

单元测试是 facades 之所以通过这样的方式工作的一个重要特性。事实上，可测试性是 facades 之所以存在的主要原因。想了解更多关于[模拟 facades](/docs/testing#mocking-facades) 内容，请参考文档的相关章节。

<a name="facade-class-reference"></a>
## Facade 类索引（Facade Class Reference）

Below you will find every facade and its underlying class. This is a useful tool for quickly digging into the API documentation for a given facade root. The [IoC binding](/docs/5.0/container) key is also included where applicable.

下面的索引中你能够找到 Laravel 提供的每一个 facade 以及所对应的真正的类。如果你想要对某个 facade 进行深入的了解，这是一个能够帮你快速找到对应 API 文档入口的工具。[IoC binding](/docs/5.0/container) 中的 key 也被列入其中。

Facade  |  Class  |  IoC Binding
------------- | ------------- | -------------
App  |  [Illuminate\Foundation\Application](http://laravel.com/api/5.0/Illuminate/Foundation/Application.html)  | `app`
Artisan  |  [Illuminate\Console\Application](http://laravel.com/api/5.0/Illuminate/Console/Application.html)  |  `artisan`
Auth  |  [Illuminate\Auth\AuthManager](http://laravel.com/api/5.0/Illuminate/Auth/AuthManager.html)  |  `auth`
Auth (Instance)  |  [Illuminate\Auth\Guard](http://laravel.com/api/5.0/Illuminate/Auth/Guard.html)  |
Blade  |  [Illuminate\View\Compilers\BladeCompiler](http://laravel.com/api/5.0/Illuminate/View/Compilers/BladeCompiler.html)  |  `blade.compiler`
Cache  |  [Illuminate\Cache\Repository](http://laravel.com/api/5.0/Illuminate/Cache/Repository.html)  |  `cache`
Config  |  [Illuminate\Config\Repository](http://laravel.com/api/5.0/Illuminate/Config/Repository.html)  |  `config`
Cookie  |  [Illuminate\Cookie\CookieJar](http://laravel.com/api/5.0/Illuminate/Cookie/CookieJar.html)  |  `cookie`
Crypt  |  [Illuminate\Encryption\Encrypter](http://laravel.com/api/5.0/Illuminate/Encryption/Encrypter.html)  |  `encrypter`
DB  |  [Illuminate\Database\DatabaseManager](http://laravel.com/api/5.0/Illuminate/Database/DatabaseManager.html)  |  `db`
DB (Instance)  |  [Illuminate\Database\Connection](http://laravel.com/api/5.0/Illuminate/Database/Connection.html)  |
Event  |  [Illuminate\Events\Dispatcher](http://laravel.com/api/5.0/Illuminate/Events/Dispatcher.html)  |  `events`
File  |  [Illuminate\Filesystem\Filesystem](http://laravel.com/api/5.0/Illuminate/Filesystem/Filesystem.html)  |  `files`
Form  |  [Illuminate\Html\FormBuilder](http://laravel.com/api/5.0/Illuminate/Html/FormBuilder.html)  |  `form`
Hash  |  [Illuminate\Hashing\HasherInterface](http://laravel.com/api/5.0/Illuminate/Hashing/HasherInterface.html)  |  `hash`
HTML  |  [Illuminate\Html\HtmlBuilder](http://laravel.com/api/5.0/Illuminate/Html/HtmlBuilder.html)  |  `html`
Input  |  [Illuminate\Http\Request](http://laravel.com/api/5.0/Illuminate/Http/Request.html)  |  `request`
Lang  |  [Illuminate\Translation\Translator](http://laravel.com/api/5.0/Illuminate/Translation/Translator.html)  |  `translator`
Log  |  [Illuminate\Log\Writer](http://laravel.com/api/5.0/Illuminate/Log/Writer.html)  |  `log`
Mail  |  [Illuminate\Mail\Mailer](http://laravel.com/api/5.0/Illuminate/Mail/Mailer.html)  |  `mailer`
Paginator  |  [Illuminate\Pagination\Factory](http://laravel.com/api/5.0/Illuminate/Pagination/Factory.html)  |  `paginator`
Paginator (Instance)  |  [Illuminate\Pagination\Paginator](http://laravel.com/api/5.0/Illuminate/Pagination/Paginator.html)  |
Password  |  [Illuminate\Auth\Passwords\PasswordBroker](http://laravel.com/api/5.0/Illuminate/Auth/Passwords/PasswordBroker.html)  |  `auth.reminder`
Queue  |  [Illuminate\Queue\QueueManager](http://laravel.com/api/5.0/Illuminate/Queue/QueueManager.html)  |  `queue`
Queue (Instance) |  [Illuminate\Queue\QueueInterface](http://laravel.com/api/5.0/Illuminate/Queue/QueueInterface.html)  |
Queue (Base Class) |  [Illuminate\Queue\Queue](http://laravel.com/api/5.0/Illuminate/Queue/Queue.html)  |
Redirect  |  [Illuminate\Routing\Redirector](http://laravel.com/api/5.0/Illuminate/Routing/Redirector.html)  |  `redirect`
Redis  |  [Illuminate\Redis\Database](http://laravel.com/api/5.0/Illuminate/Redis/Database.html)  |  `redis`
Request  |  [Illuminate\Http\Request](http://laravel.com/api/5.0/Illuminate/Http/Request.html)  |  `request`
Response  |  [Illuminate\Support\Facades\Response](http://laravel.com/api/5.0/Illuminate/Support/Facades/Response.html)  |
Route  |  [Illuminate\Routing\Router](http://laravel.com/api/5.0/Illuminate/Routing/Router.html)  |  `router`
Schema  |  [Illuminate\Database\Schema\Blueprint](http://laravel.com/api/5.0/Illuminate/Database/Schema/Blueprint.html)  |
Session  |  [Illuminate\Session\SessionManager](http://laravel.com/api/5.0/Illuminate/Session/SessionManager.html)  |  `session`
Session (Instance)  |  [Illuminate\Session\Store](http://laravel.com/api/5.0/Illuminate/Session/Store.html)  |
SSH  |  [Illuminate\Remote\RemoteManager](http://laravel.com/api/5.0/Illuminate/Remote/RemoteManager.html)  |  `remote`
SSH (Instance)  |  [Illuminate\Remote\Connection](http://laravel.com/api/5.0/Illuminate/Remote/Connection.html)  |
URL  |  [Illuminate\Routing\UrlGenerator](http://laravel.com/api/5.0/Illuminate/Routing/UrlGenerator.html)  |  `url`
Validator  |  [Illuminate\Validation\Factory](http://laravel.com/api/5.0/Illuminate/Validation/Factory.html)  |  `validator`
Validator (Instance)  |  [Illuminate\Validation\Validator](http://laravel.com/api/5.0/Illuminate/Validation/Validator.html) |
View  |  [Illuminate\View\Factory](http://laravel.com/api/5.0/Illuminate/View/Factory.html)  |  `view`
View (Instance)  |  [Illuminate\View\View](http://laravel.com/api/5.0/Illuminate/View/View.html)  |
