# 配置（Configuration）

- [简介（Introduction）](#introduction)
- [完成安装后（After Installation）](#after-installation)
- [获取当前应用的环境（Accessing Configuration Values）](#accessing-configuration-values)
- [环境配置（Environment Configuration）](#environment-configuration)
- [缓存配置（Configuration Caching）](#configuration-caching)
- [维护模式（Maintenance Mode）](#maintenance-mode)
- [URL 美化（Pretty URLs）](#pretty-urls)

<a name="introduction"></a>
## 简介（Introduction）

Laravel 框架的所有配置文件都存放在 `config` 目录中。每个选项都有对应的描述文档，所以你可以通过阅读这些配置文件对这些配置项更加熟悉。

<a name="after-installation"></a>
## 完成安装后（After Installation）

### 对你的应用命名（Naming Your Application）

完成 Laravel 的安装后，你可能希望给你的应用起个名字。默认情况下，`app` 目录被设置成名为 `App` 的命名空间，并且会被 Composer 使用 [PSR-4 自动加载标准](http://www.php-fig.org/psr/psr-4/) 进行自动加载。但是，你可以通过 `app:name` 这个 Artisan 命令很容易的对这个命名空间进行修改，使之与你的应用相匹配。

例如，如果你的应用名是 “Horsefly”，你可以在安装框架的根目录下运行下面这条命令：

	php artisan app:name Horsefly

对应用进行重命名完全是可选的，如果愿意的话完全可以直接使用 `App` 这个命名空间。

### 其他配置（Other Configuration）

Laravel 安装完成后只需要进行非常少量的配置就可以使用了。但是，你仍然需要浏览一遍 `config/app.php` 这个配置文件以及相关的文档。它包含了一些与地区或时间相关的配置，你可能希望据实际情况对他们进行一些相应的调整，例如 `timezone` 和 `locale`。

Laravel 安装完成后，你还需要 [配置你本地的开发环境](/docs/5.0/configuration#environment-configuration)。

> **注意：** 一定不要把生产环境中的应用的 `app.debug` 配置项设置为 `ture`。

<a name="permissions"></a>
### 权限许可（Permissions）

还需要对 Laravel 进行一些权限的设置：web server 需要对 `storage` 中的目录有写入的权限。

<a name="accessing-configuration-values"></a>
## 读取配置文件（Accessing Configuration Values）

你可以使用 `Config` facade 很方便的获取配置文件的属性值：

	$value = Config::get('app.timezone');

	Config::set('app.timezone', 'America/Chicago');

也可以使用 `config` 辅助函数实现同样的功能：

	$value = config('app.timezone');

<a name="environment-configuration"></a>
## 环境配置（Environment Configuration）

能够针对应用程序所运行的环境的不同来设置不同的配置是很有用的。例如，你在本地开发环境和生产环境中可能希望使用不同的缓存驱动。它可以通过环境配置很容易实现。

Laravel 利用了 Vance Lucas 的 [DotEnv](https://github.com/vlucas/phpdotenv) PHP 类库来达到这个目的。在一个刚安装好的 Laravel 应用的根目录下应该有一个 `.env.example` 文件。如果你通过 Composer 安装的话，这个文件会自动被改名为 `.env`。否则，你需要手动修改这个文件的名字。

当应用收到一个请求后，这个文件中所有的变量都将被载入到 PHP 的 `$_ENV` 超级全局变量中。你可以使用 `env` 辅助函数获取这些变量的值。如果你看过 Laravel 的配置文件，你会发现有些选项已经在使用这个辅助函数了。

你可以根据你本地服务器的需要任意修改你的环境配置变量，生产环境也是如此。但是，你的 `.evn` 文件不应该被提交到应用的版本控制中去，因为每个使用同一个应用的开发者或者服务器都需要不同的环境配置。

如果你是在一个团队中进行开发，你可能希望在应用中一直包含一个 `.env.example` 的文件。通过在这个示例配置文件中插入占位值，你团队里的其他成员可以很清晰的看到应用运行时需要哪些环境变量。

#### 获取当前应用的环境（Accessing The Current Application Environment）

可以通过 `Application` 实例的 `environment` 方法获取当前应用所处的运行环境：

	$environment = $app->environment();

你也可以给 `environment` 方法传入参数来检查当前运行环境是否和你出入的值一致：

	if ($app->environment('local'))
	{
		// The environment is local
	}

	if ($app->environment('local', 'staging'))
	{
		// The environment is either local OR staging...
	}

想要获得应用的实例，可以通过 [容器](/docs/5.0/container) 从 `Illuminate\Contracts\Foundation\Application` 的合约类中获得。当然，如果正处于一个 [service provider](/docs/5.0/providers) 中，也可以通过 `$this->app` 这个实例变量来获得。

应用的实例也可以通过 `app` 辅助方法或 `App` facade 来访问：

	$environment = app()->environment();

	$environment = App::environment();

<a name="configuration-caching"></a>
## 缓存配置（Configuration Caching）

来给你的应用提速吧，可以通过 `config:cache` Artisan 命令把所有的配置文件都缓存到一个文件中。它会把你的应用用到的所有配置项，合并后放入一个可以被框架快速加载的文件中。

你应该把运行 `config:cache` 命令，作为你对应用进行部署的一个环节。

<a name="maintenance-mode"></a>
## 维护模式（Maintenance Mode）

当你的应用处于维护模式时，每一个到达应用的请求都会被返回并显示一个自定义的页面。它可以使你在升级或者进行维护时，很容易的将应用设置为“不可用”状态。对维护模式的检查是位于应用的默认中间件堆栈中。如果应用处于维护模式下，会抛出一个状态码为 503 的 `HttpException`。

可以通过执行 `down` Artisan 命令快速开启维护模式：

	php artisan down

使用 `up` 命令关闭维护模式：

	php artisan up

### 维护模式返回的模板（Maintenance Mode Response Template）

维护模式默认返回的模板是位于 `resources/views/errors/503.blade.php` 的视图文件。

### 维护模式与队列（Maintenance Mode & Queues）

当应用处于维护状态时，[队列任务](/docs/5.0/queues) 不会被之情。未完成的队列任务会在应用从维护模式恢复到正常状态后继续执行。

<a name="pretty-urls"></a>
## URL 美化（Pretty URLs）

### Apache

框架自带了 `public/.htaccess` 文件，用于允许 URL 中不使用 `index.php` 。如果你是使用 Apache 作为 Laravel 的 web server，请确保已经开启了 `mod_rewrite` 模块。

如果 Laravel 自带的 `.htaccess` 文件在你的 Apache 下不起作用，请尝试一下这个规则：

	Options +FollowSymLinks
	RewriteEngine On

	RewriteCond %{REQUEST_FILENAME} !-d
	RewriteCond %{REQUEST_FILENAME} !-f
	RewriteRule ^ index.php [L]

### Nginx

在 Nginx 下，请在你网站的配置中使用以下的重定向规则，来实现 URL 美化：

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

当然，如果使用 [Homestead](/docs/5.0/homestead) ，URL 美化会自动配置好的。
