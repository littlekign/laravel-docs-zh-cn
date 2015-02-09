# 安装（Installation）

- [安装 Composer（Install Composer）](#install-composer)
- [安装 Laravel（Install Laravel）](#install-laravel)
- [服务器环境要求（Server Requirements）](#server-requirements)

<a name="install-composer"></a>
## 安装 Composer（Install Composer）

Laravel 框架利用 [Composer](http://getcomposer.org) 进行依赖管理。所以在使用 Laravel 前，你需要确保已经在你的机器上安装了 Composer。

<a name="install-laravel"></a>
## 安装 Laravel（Install Laravel）

### 通过 Laravel 安装程序进行安装（Via Laravel Installer）

首先，使用 Composer 下载 Laravel 的安装程序。

	composer global require "laravel/installer=~1.1"

请确保已经把 `~/.composer/vendor/bin` 目录添加到你系统的环境变量 PATH 中，这样你的系统才能定位到 `laravel` 这个可执行程序。

安装完成后，就可以使用 `laravel new` 命令在你指定的目录下创建一个全新的 Laravel 应用了。例如，执行 `laravel new blog` 命令，就会创建一个名为 `blog` 的目录，里面包含一个全新的 Laravel 框架以及所有依赖的程序。用这种方法来安装要比通过 Composer 安装快很多：

	laravel new blog

### 通过 Composer Create-Project 安装（Via Composer Create-Project）

你也可以在终端输入 Composer 的 `create-project` 命令来安装 Laravel。

	composer create-project laravel/laravel --prefer-dist

<a name="server-requirements"></a>
## 服务器环境要求（Server Requirements）

Laravel 框架对系统环境有一些新的要求：

- PHP 版本要大于等于 5.4
- 安装 PHP 的 Mcrypt 扩展
- 安装 PHP 的 OpenSSL 扩展
- 安装 PHP 的 Mbstring 扩展

对于 PHP 5.5 来说，在某些系统的发行版中可能需要你手动安装 JSON 扩展。使用 Ubuntu 的话，可以通过 `apt-get install php5-json` 来安装。

<a name="configuration"></a>
## 配置（Configuration）

安装完 Laravel 框架后需要做的第一件事是给你的应用设置一个随机的字符串作为 key。如果你是通过 Composer 来安装的，这个 key 应该已经通过 `key:generate` 这个命令设置好了。

一般来说，它应该是一个 32 个字符长度的字符串。可以在 `.env` 环境配置文件中进行设置。**如果没有给应用设置 key，你的用户的 session 以及其他需要加密的数据都将是不安全的！**

除了这个 Laravel 几乎不需要再进行其他配置了。你已经可以开始进行开发了！但是，你可能希望先去看一下 `config/app.php` 文件以及相关的文档。它包含许多类似于 `timezone` 和 `locale` 这样的可配置项，你可以根据你的应用的需要对他们进行配置。

Laravel 安装完成后，你还需要 [配置你本地的开发环境](/docs/5.0/configuration#environment-configuration)

> **注意：** 一定不要把生产环境中的应用的 `app.debug` 配置项设置为 `ture`。

<a name="permissions"></a>
### 权限许可（Permissions）

还需要对 Laravel 进行一些权限的设置：web server 需要对 `storage` 中的目录有写入的权限。

<a name="pretty-urls"></a>
## URL美化（Pretty URLs）

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
