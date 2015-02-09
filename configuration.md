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

All of the configuration files for the Laravel framework are stored in the `config` directory. Each option is documented, so feel free to look through the files and get familiar with the options available to you.

Laravel 框架的所有配置文件都存放在 `config` 目录中。每个选项都有对应的描述文档，所以你可以通过阅读这些配置文件对这些配置项更加熟悉。

<a name="after-installation"></a>
## 完成安装后（After Installation）

### 对你的应用命名（Naming Your Application）

After installing Laravel, you may wish to "name" your application. By default, the `app` directory is namespaced under `App`, and autoloaded by Composer using the [PSR-4 autoloading standard](http://www.php-fig.org/psr/psr-4/). However, you may change the namespace to match the name of your application, which you can easily do via the `app:name` Artisan command.

完成 Laravel 的安装后，你可能希望给你的应用起个名字。默认情况下，`app` 目录被设置成名为 `App` 的命名空间，并且会被 Composer 使用 [PSR-4 自动加载标准](http://www.php-fig.org/psr/psr-4/) 进行自动加载。但是，你可以通过 `app:name` 这个 Artisan 命令很容易的对这个命名空间进行修改，使之与你的应用相匹配。

For example, if your application is named "Horsefly", you could run the following command from the root of your installation:

例如，如果你的应用名是 “Horsefly”，你可以在安装框架的根目录下运行下面这条命令：

	php artisan app:name Horsefly

Renaming your application is entirely optional, and you are free to keep the `App` namespace if you wish.

对应用进行重命名完全是可选的，如果愿意的话完全可以直接使用 `App` 这个命名空间。

### 其他配置（Other Configuration）

Laravel needs very little configuration out of the box. You are free to get started developing! However, you may wish to review the `config/app.php` file and its documentation. It contains several options such as `timezone` and `locale` that you may wish to change according to your location.

Laravel 安装完成后只需要进行非常少量的配置就可以使用了。但是，你仍然需要浏览一遍 `config/app.php` 这个配置文件以及相关的文档。它包含了一些与地区或时间相关的配置，你可能希望据实际情况对他们进行一些相应的调整，例如 `timezone` 和 `locale`。

Once Laravel is installed, you should also [configure your local environment](/docs/5.0/configuration#environment-configuration).

Laravel 安装完成后，你还需要 [配置你本地的开发环境](/docs/5.0/configuration#environment-configuration)。

> **Note:** You should never have the `app.debug` configuration option set to `true` for a production application.

> **注意：** 一定不要把生产环境中的应用的 `app.debug` 配置项设置为 `ture`。

<a name="permissions"></a>
### 权限许可（Permissions）

Laravel may require one set of permissions to be configured: folders within `storage` require write access by the web server.

还需要对 Laravel 进行一些权限的设置：web server 需要对 `storage` 中的目录有写入的权限。

<a name="accessing-configuration-values"></a>
## 读取配置文件（Accessing Configuration Values）

You may easily access your configuration values using the `Config` facade:

你可以使用 `Config` facade 很方便的获取配置文件的属性值：

	$value = Config::get('app.timezone');

	Config::set('app.timezone', 'America/Chicago');

You may also use the `config` helper function:

也可以使用 `config` 辅助函数实现同样的功能：

	$value = config('app.timezone');

<a name="environment-configuration"></a>
## 环境配置（Environment Configuration）

It is often helpful to have different configuration values based on the environment the application is running in. For example, you may wish to use a different cache driver locally than you do on your production server. It's easy using environment based configuration.

能够针对应用程序所运行的环境的不同来设置不同的配置是很有用的。例如，你在本地开发环境和生产环境中可能希望使用不同的缓存驱动。它可以通过环境配置很容易实现。

To make this a cinch, Laravel utilizes the [DotEnv](https://github.com/vlucas/phpdotenv) PHP library by Vance Lucas. In a fresh Laravel installation, the root directory of your application will contain a `.env.example` file. If you install Laravel via Composer, this file will automatically be renamed to `.env`. Otherwise, you should rename the file manually.

Laravel 利用了 Vance Lucas 的 [DotEnv](https://github.com/vlucas/phpdotenv) PHP 类库来达到这个目的。在一个刚安装好的 Laravel 应用的根目录下应该有一个 `.env.example` 文件。如果你通过 Composer 安装的话，这个文件会自动被改名为 `.env`。否则，你需要手动修改这个文件的名字。

All of the variables listed in this file will be loaded into the `$_ENV` PHP super-global when your application receives a request. You may use the `env` helper to retrieve values from these variables. In fact, if you review the Laravel configuration files, you will notice several of the options already using this helper!

当应用收到一个请求后，这个文件中所有的变量都将被载入到 PHP 的 `$_ENV` 超级全局变量中。你可以使用 `env` 辅助函数获取这些变量的值。如果你看过 Laravel 的配置文件，你会发现有些选项已经在使用这个辅助函数了。

Feel free to modify your environment variables as needed for your own local server, as well as your production environment. However, your `.env` file should not be committed to your application's source control, since each developer / server using your application could require a different environment configuration.

你可以根据你本地服务器的需要任意修改你的环境配置变量，生产环境也是如此。但是，你的 `.evn` 文件不应该被提交到应用的版本控制中去，因为每个使用同一个应用的开发者或者服务器都需要不同的环境配置。

If you are developing with a team, you may wish to continue including a `.env.example` file with your application. By putting place-holder values in the example configuration file, other developers on your team can clearly see which environment variables are needed to run your application.

如果你是在一个团队中进行开发，你可能希望在应用中一直包含一个 `.env.example` 的文件。通过在这个示例配置文件中插入占位值，你团队里的其他成员可以很清晰的看到应用运行时需要哪些环境变量。

#### 获取当前应用的环境（Accessing The Current Application Environment）

You may access the current application environment via the `environment` method on the `Application` instance:

可以通过 `Application` 实例的 `environment` 方法获取当前应用所处的运行环境：

	$environment = $app->environment();

You may also pass arguments to the `environment` method to check if the environment matches a given value:

你也可以给 `environment` 方法传入参数来检查当前运行环境是否和你出入的值一致：

	if ($app->environment('local'))
	{
		// The environment is local
	}

	if ($app->environment('local', 'staging'))
	{
		// The environment is either local OR staging...
	}

To obtain an instance of the application, resolve the `Illuminate\Contracts\Foundation\Application` contract via the [service container](/docs/5.0/container). Of course, if you are within a [service provider](/docs/5.0/providers), the application instance is available via the `$this->app` instance variable.

想要获得应用的实例，可以通过 [容器](/docs/5.0/container) 从 `Illuminate\Contracts\Foundation\Application` 的合约类中获得。当然，如果正处于一个 [service provider](/docs/5.0/providers) 中，也可以通过 `$this->app` 这个实例变量来获得。

An application instance may also be accessed via the `app` helper of the `App` facade:

应用的实例也可以通过 `app` 辅助方法或 `App` facade 来访问：

	$environment = app()->environment();

	$environment = App::environment();

<a name="configuration-caching"></a>
## 缓存配置（Configuration Caching）

To give your application a little speed boost, you may cache all of your configuration files into a single file using the `config:cache` Artisan command. This will combine all of the configuration options for your application into a single file which can be loaded quickly by the framework.

来给你的应用提速吧，可以通过 `config:cache` Artisan 命令把所有的配置文件都缓存到一个文件中。它会把你的应用用到的所有配置项，合并后放入一个可以被框架快速加载的文件中。

You should typically run the `config:cache` command as part of your deployment routine.

你应该把运行 `config:cache` 命令，作为你对应用进行部署的一个环节。

<a name="maintenance-mode"></a>
## 维护模式（Maintenance Mode）

When your application is in maintenance mode, a custom view will be displayed for all requests into your application. This makes it easy to "disable" your application while it is updating or when you are performing maintenance. A maintenance mode check is included in the default middleware stack for your application. If the application is in maintenance mode, an `HttpException` will be thrown with a status code of 503.

当你的应用处于维护模式时，每一个到达应用的请求都会被返回并显示一个自定义的页面。它可以使你在升级或者进行维护时，很容易的将应用设置为“不可用”状态。对维护模式的检查是位于应用的默认中间件堆栈中。如果应用处于维护模式下，会抛出一个状态码为 503 的 `HttpException`。

To enable maintenance mode, simply execute the `down` Artisan command:

可以通过执行 `down` Artisan 命令快速开启维护模式：

	php artisan down

To disable maintenance mode, use the `up` command:

使用 `up` 命令关闭维护模式：

	php artisan up

### 维护模式返回的模板（Maintenance Mode Response Template）

The default template for maintenance mode responses is located in `resources/views/errors/503.blade.php`.

维护模式默认返回的模板是位于 `resources/views/errors/503.blade.php` 的视图文件。

### 维护模式与队列（Maintenance Mode & Queues）

While your application is in maintenance mode, no [queued jobs](/docs/5.0/queues) will be handled. The jobs will continue to be handled as normal once the application is out of maintenance mode.

当应用处于维护状态时，[队列任务](/docs/5.0/queues) 不会被之情。未完成的队列任务会在应用从维护模式恢复到正常状态后继续执行。

<a name="pretty-urls"></a>
## URL 美化（Pretty URLs）

### Apache

The framework ships with a `public/.htaccess` file that is used to allow URLs without `index.php`. If you use Apache to serve your Laravel application, be sure to enable the `mod_rewrite` module.

框架自带了 `public/.htaccess` 文件，用于允许 URL 中不使用 `index.php` 。如果你是使用 Apache 作为 Laravel 的 web server，请确保已经开启了 `mod_rewrite` 模块。

If the `.htaccess` file that ships with Laravel does not work with your Apache installation, try this one:

如果 Laravel 自带的 `.htaccess` 文件在你的 Apache 下不起作用，请尝试一下这个规则：

	Options +FollowSymLinks
	RewriteEngine On

	RewriteCond %{REQUEST_FILENAME} !-d
	RewriteCond %{REQUEST_FILENAME} !-f
	RewriteRule ^ index.php [L]

### Nginx

On Nginx, the following directive in your site configuration will allow "pretty" URLs:

在 Nginx 下，请在你网站的配置中使用以下的重定向规则，来实现 URL 美化：

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

Of course, when using [Homestead](/docs/5.0/homestead), pretty URLs will be configured automatically.

当然，如果使用 [Homestead](/docs/5.0/homestead) ，URL 美化会自动配置好的。
