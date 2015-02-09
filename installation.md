# 安装（Installation）

- [安装 Composer（Install Composer）](#install-composer)
- [安装 Laravel（Install Laravel）](#install-laravel)
- [服务器环境要求（Server Requirements）](#server-requirements)

<a name="install-composer"></a>
## 安装 Composer（Install Composer）

Laravel utilizes [Composer](http://getcomposer.org) to manage its dependencies. So, before using Laravel, you will need to make sure you have Composer installed on your machine.

Laravel 框架利用 [Composer](http://getcomposer.org) 进行依赖管理。所以在使用 Laravel 前，你需要确保已经在你的机器上安装了 Composer。

<a name="install-laravel"></a>
## 安装 Laravel（Install Laravel）

### 通过 Laravel 安装程序进行安装（Via Laravel Installer）

First, download the Laravel installer using Composer.

首先，使用 Composer 下载 Laravel 的安装程序。

	composer global require "laravel/installer=~1.1"

Make sure to place the `~/.composer/vendor/bin` directory in your PATH so the `laravel` executable can be located by your system.

请确保已经把 `~/.composer/vendor/bin` 目录添加到你系统的环境变量 PATH 中，这样你的系统才能定位到 `laravel` 这个可执行程序。

Once installed, the simple `laravel new` command will create a fresh Laravel installation in the directory you specify. For instance, `laravel new blog` would create a directory named `blog` containing a fresh Laravel installation with all dependencies installed. This method of installation is much faster than installing via Composer:

安装完成后，就可以使用 `laravel new` 命令在你指定的目录下创建一个全新的 Laravel 应用了。例如，执行 `laravel new blog` 命令，就会创建一个名为 `blog` 的目录，里面包含一个全新的 Laravel 框架以及所有依赖的程序。用这种方法来安装要比通过 Composer 安装快很多：

	laravel new blog

### 通过 Composer Create-Project 安装（Via Composer Create-Project）

You may also install Laravel by issuing the Composer `create-project` command in your terminal:

你也可以在终端输入 Composer 的 `create-project` 命令来安装 Laravel。

	composer create-project laravel/laravel --prefer-dist

<a name="server-requirements"></a>
## 服务器环境要求（Server Requirements）

The Laravel framework has a few system requirements:

Laravel 框架对系统环境有一些新的要求：

- PHP >= 5.4
- Mcrypt PHP Extension
- OpenSSL PHP Extension
- Mbstring PHP Extension

- PHP 版本要大于等于 5.4
- 安装 PHP 的 Mcrypt 扩展
- 安装 PHP 的 OpenSSL 扩展
- 安装 PHP 的 Mbstring 扩展

As of PHP 5.5, some OS distributions may require you to manually install the PHP JSON extension. When using Ubuntu, this can be done via `apt-get install php5-json`.

对于 PHP 5.5 来说，在某些系统的发行版中可能需要你手动安装 JSON 扩展。使用 Ubuntu 的话，可以通过 `apt-get install php5-json` 来安装。

<a name="configuration"></a>
## 配置（Configuration）

The first thing you should do after installing Laravel is set your application key to a random string. If you installed Laravel via Composer, this key has probably already been set for you by the `key:generate` command.

安装完 Laravel 框架后需要做的第一件事是给你的应用设置一个随机的字符串作为 key。如果你是通过 Composer 来安装的，这个 key 应该已经通过 `key:generate` 这个命令设置好了。

Typically, this string should be 32 characters long. The key can be set in the `.env` environment file. **If the application key is not set, your user sessions and other encrypted data will not be secure!**

一般来说，它应该是一个 32 个字符长度的字符串。可以在 `.env` 环境配置文件中进行设置。**如果没有给应用设置 key，你的用户的 session 以及其他需要加密的数据都将是不安全的！**

Laravel needs almost no other configuration out of the box. You are free to get started developing! However, you may wish to review the `config/app.php` file and its documentation. It contains several options such as `timezone` and `locale` that you may wish to change according to your application.

除了这个 Laravel 几乎不需要再进行其他配置了。你已经可以开始进行开发了！但是，你可能希望先去看一下 `config/app.php` 文件以及相关的文档。它包含许多类似于 `timezone` 和 `locale` 这样的可配置项，你可以根据你的应用的需要对他们进行配置。

Once Laravel is installed, you should also [configure your local environment](/docs/5.0/configuration#environment-configuration).

Laravel 安装完成后，你还需要 [配置你本地的开发环境](/docs/5.0/configuration#environment-configuration)

> **Note:** You should never have the `app.debug` configuration option set to `true` for a production application.

> **注意：** 一定不要把生产环境中的应用的 `app.debug` 配置项设置为 `ture`。

<a name="permissions"></a>
### 权限许可（Permissions）

Laravel may require some permissions to be configured: folders within `storage` require write access by the web server.

还需要对 Laravel 进行一些权限的设置：web server 需要对 `storage` 中的目录有写入的权限。

<a name="pretty-urls"></a>
## URL美化（Pretty URLs）

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
