# 容器（Service Container）

- [简介（Introduction）](#introduction)
- [基本用法（Basic Usage）](#basic-usage)
- [接口与实例的绑定（Binding Interfaces To Implementations）](#binding-interfaces-to-implementations)
- [按需绑定（Contextual Binding）](#contextual-binding)
- [标签（Tagging）](#tagging)
- [实际应用（Practical Applications）](#practical-applications)
- [容器事件（Container Events）](#container-events)

<a name="introduction"></a>
## 简介（Introduction）

Laravel 的容器（service container）是一个非常强大的用来管理类依赖关系的工具。依赖注入这个有点梦幻的词在这里的实际意义是：某个类通过自身的构造函数或“setter”方法将这个类所依赖于其他类的依赖关系表示出来。

我们来看个例子：

	<?php namespace App\Handlers\Commands;

	use App\User;
	use App\Commands\PurchasePodcast;
	use Illuminate\Contracts\Mail\Mailer;

	class PurchasePodcastHandler {

		/**
		 * The mailer implementation.
		 */
		protected $mailer;

		/**
		 * Create a new instance.
		 *
		 * @param  Mailer  $mailer
		 * @return void
		 */
		public function __construct(Mailer $mailer)
		{
			$this->mailer = $mailer;
		}

		/**
		 * Purchase a podcast.
		 *
		 * @param  PurchasePodcastCommand  $command
		 * @return void
		 */
		public function handle(PurchasePodcastCommand $command)
		{
			//
		}

	}

在这个例子中，`PurchasePodcast` 处理类，需要当一个播客节目被支付时发送邮件。所以，我们需要将一个能够发送邮件的服务 **注入** 进来。因为这个服务是被注入进来的，所以我们能够很轻易地用另一个其他的服务来替换它。也可以在测试应用的时候，很简便地“伪造”或创建一个仿冒的 mailer 实例。

想要对 Laravel 的容器（service container）有更深入的理解，就需要从基础去构建一个复杂的大型应用，例如为 Laravel 框架本身贡献代码。

<a name="basic-usage"></a>
## 基本用法（Basic Usage）

### 绑定

基本上所有的容器绑定都需要在 [service providers](/docs/5.0/providers) 中进行注册，所以下面所有的示例都将模拟在这种环境下使用容器。如果你在程序的任何地方（例如某个工厂类中）需要使用容器时，你都可以通过类型约束（type-hint）来指定一个约定类 `Illuminate\Contracts\Container\Container` ，同时能够得到被注入的容器的实例。另外，也可以通过 `App` 来操作容器。

#### 注册一个基础的 Resolver

在一个 service provider 中，可以一直通过 `$this->app` 实例变量来操作容器。

有几种方法可以给容器注册依赖关系，包括闭包回调和绑定接口到实例的方法。先看一下闭包回调的方法。在容器中注册一个闭包 resolver，需要传一个 key（一般是类名）以及一个返回一些数据的闭包函数：

	$this->app->bind('FooBar', function($app)
	{
		return new FooBar($app['SomethingElse']);
	});

#### 注册单例方法

有时，我们需要将一些只允许执行一次的罗技绑定到容器中，在接下来的调用中，将始终返回第一次调用时生成的实例：

	$this->app->singleton('FooBar', function($app)
	{
		return new FooBar($app['SomethingElse']);
	});

#### 绑定一个已经存在的实例到容器

也可以使用`instance`方法绑定一个已经存在的类的实例到容器中。在后续对容器调用中将始终返回该实例：


	$fooBar = new FooBar(new SomethingElse);

	$this->app->instance('FooBar', $fooBar);

### 调用（Resolving）

有几种方法可以从容器中找到之前注册在里面的东西。首先，可以用 `make` 方法：

	$fooBar = $this->app->make('FooBar');

其次，可以通过访问数组的方式读取容器，因为容器实现了 PHP 的 `ArrayAccess` 接口：

	$fooBar = $this->app['FooBar'];

最后，但也是最重要的是，你可以简单地通过“类型约定”对那些通过容器调取的类，的构造方法设置依赖关系，包括控制器，事件监听器，队列任务，过滤器，以及其他。容器将会自动将依赖注入：

	<?php namespace App\Http\Controllers;

	use Illuminate\Routing\Controller;
	use App\Users\Repository as UserRepository;

	class UserController extends Controller {

		/**
		 * The user repository instance.
		 */
		protected $users;

		/**
		 * Create a new controller instance.
		 *
		 * @param  UserRepository  $users
		 * @return void
		 */
		public function __construct(UserRepository $users)
		{
			$this->users = $users;
		}

		/**
		 * Show the user with the given ID.
		 *
		 * @param  int  $id
		 * @return Response
		 */
		public function show($id)
		{
			//
		}

	}

<a name="binding-interfaces-to-implementations"></a>
## 接口与实例绑定

### 依赖实体注入

容器拥有一个非常强大的功能，它可以将一个接口绑定到一个指定的实例上。例如，我们的应用需要与 [Pusher](https://pusher.com) 这个推送服务交互，来发送和接收实时消息。如果我们正在用 Pusher 的 PHP SDK，就可以将 Pusher 客户端的一个实例注入到一个类中：

	<?php namespace App\Handlers\Commands;

	use App\Commands\CreateOrder;
	use Pusher\Client as PusherClient;

	class CreateOrderHandler {

		/**
		 * The Pusher SDK client instance.
		 */
		protected $pusher;

		/**
		 * Create a new order handler instance.
		 *
		 * @param  PusherClient  $pusher
		 * @return void
		 */
		public function __construct(PusherClient $pusher)
		{
			$this->pusher = $pusher;
		}

		/**
		 * Execute the given command.
		 *
		 * @param  CreateOrder  $command
		 * @return void
		 */
		public function execute(CreateOrder $command)
		{
			//
		}

	}

在这个示例中，它的好处是将这个类的依赖注入了进来；但同时，我们也把它和 Pusher 的 SDK 紧紧捆绑在了一起。如果以后 Pusher 的 SDK 发生了变化或者我们打算全部切换到一个新的服务上去，就需要对 `CreateOrderHandler` 的代码进行修改。

### 对接口编码（Program To An Interface）

为了将 `CreateOrderHandler` 与可能变化的事件服务进行“隔离”，可以定义一个 `EventPusher` 的接口及一个 `PusherEventPusher` 的推送服务实例：

	<?php namespace App\Contracts;

	interface EventPusher {

		/**
		 * Push a new event to all clients.
		 *
		 * @param  string  $event
		 * @param  array  $data
		 * @return void
		 */
		public function push($event, array $data);

	}

当我们将这个接口的实例 `PusherEventPusher` 编写好，就可以通过容器对它进行注册：

	$this->app->bind('App\Contracts\EventPusher', 'App\Services\PusherEventPusher');

上面的代码告诉容器，当一个类需要一个 `EventPusher` 的实例时，容器应当将 `PusherEventPusher` 注入到这个类中。现在我们可以通过类型约定把 `EventPusher` 这个接口绑定到构造函数上：

		/**
		 * Create a new order handler instance.
		 *
		 * @param  EventPusher  $pusher
		 * @return void
		 */
		public function __construct(EventPusher $pusher)
		{
			$this->pusher = $pusher;
		}

<a name="contextual-binding"></a>
## 按需绑定

有的时候可能会有两个类需要调用同一个接口，但又希望对这两个类注入不同的实例。例如，当系统接收到一个新订单，我们相通过 [PubNub](http://www.pubnub.com/) 服务发送通知而不是 Pusher。Laravel 提供了一个简单，流畅的接口用来定义这种行为：

	$this->app->when('App\Handlers\Commands\CreateOrderHandler')
	          ->needs('App\Contracts\EventPusher')
	          ->give('App\Services\PubNubEventPusher');

<a name="tagging"></a>
## 标签（Tagging）

偶尔，可能会需要使用某一类别的所有绑定。假如，你在编写一个报告聚合器，它会收到一个包含很多中不同的 `Report` 接口的实例。在完成对所有 `Report` 实例的注册后，可以通过 `tag` 方法给他们打上一个标签：

	$this->app->bind('SpeedReport', function()
	{
		//
	});

	$this->app->bind('MemoryReport', function()
	{
		//
	});

	$this->app->tag(['SpeedReport', 'MemoryReport'], 'reports');

当所有的服务都被打上标签后，就可以通过 `tagged` 方法很轻松的一次性全部调用他们：

	$this->app->bind('ReportAggregator', function($app)
	{
		return new ReportAggregator($app->tagged('reports'));
	});

<a name="practical-applications"></a>
## 实际应用（Practical Applications）

Laravel 提供很多使用容器来增加程序的灵活性和可测试性的机会。最典型的例子就是使用控制器的时候。所有的控制器都会通过容器来调用，这就意味着你可以在控制器的构造方法中通过类型约束绑定类的依赖，这些依赖的类会被自动注入。

	<?php namespace App\Http\Controllers;

	use Illuminate\Routing\Controller;
	use App\Repositories\OrderRepository;

	class OrdersController extends Controller {

		/**
		 * The order repository instance.
		 */
		protected $orders;

		/**
		 * Create a controller instance.
		 *
		 * @param  OrderRepository  $orders
		 * @return void
		 */
		public function __construct(OrderRepository $orders)
		{
			$this->orders = $orders;
		}

		/**
		 * Show all of the orders.
		 *
		 * @return Response
		 */
		public function index()
		{
			$all = $this->orders->all();

			return view('orders', ['all' => $all]);
		}

	}

在这个例子中，`OrderRepository` 类会被自动注入到控制器中。这也意味着可以在进行[单元测试](/docs/5.0/testing)时将一个模拟的 `OrderRepository` 绑定到容器中，以此来实现与数据层的无痛交互。

#### 容器的其他用法（Other Examples Of Container Usage）

当然，像之前提到的，控制器不是 Laravel 唯一通过容器来调用的类。你同样可以在 route Closures，filters，queue jobs, event listeners 等其他的 Laravel 类中进行依赖关系约定。请跳转到文档的对应章节查看通过容器使用这些内容的具体示例。

<a name="container-events"></a>
## 容器事件（Container Events）

#### 调用监听器的注册（Registering A Resolving Listener）

容器会在每一次调用一个对象时触发一个事件。你可以使用 `resolving` 方法来监听这类事件：

	$this->app->resolving(function($object, $app)
	{
		// Called when container resolves object of any type...
	});

	$this->app->resolving(function(FooBar $fooBar, $app)
	{
		// Called when container resolves objects of type "FooBar"...
	});

被调用过的对象将被传递给回调函数。
