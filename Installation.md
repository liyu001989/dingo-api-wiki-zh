To install this package you will need:

要安装这个包，你需要：

- Laravel 5.1+ or Lumen 5.1+
- PHP 5.5.9+

You must then modify your `composer.json` file and run `composer update` to include the latest version of the package in your project.

你需要修改你的 `composer.json` 文件，然后执行 `composer update` 把最后一个版本的包加入你的项目

```json
"require": {
    "dingo/api": "1.0.*@dev"
}
```

Or you can run the `composer require` command from your terminal.

或者你可以在命令行执行 `composer require` 命令

```
composer require dingo/api:1.0.x@dev
```

> At this time the package is still in a developmental stage and as such does not have a **stable** release.
> You may need to set your `minimum-stability` to `dev`.

> 现在这个包还在开发中，还没有 **stable** 版本。
> 你可能需要设置你的 `minimum-stability` 为 `dev`。

Once the package is installed the next step is dependant on which framework you're using.

一旦这个包被安装上，下一步就取决于你使用的是哪个框架

### Laravel

Open `config/app.php` and register the required service provider **above** your application providers.

打开 `config/app.php`，注册必要的 service provider 在你的应用 providers **之前**。

```php
'providers' => [
    Dingo\Api\Provider\LaravelServiceProvider::class
]
```

If you'd like to make configuration changes in the configuration file you can pubish it with the following Aritsan command:

如果你想在配置文件中改变一些配置，你可以使用下面的 Artisan 命令发布配置文件

```
php artisan vendor:publish --provider="Dingo\Api\Provider\LaravelServiceProvider"
```

### Lumen

Open `bootstrap/app.php` and register the required service provider.

打开 `bootstrap/app.php`，注册必要的 service providers

```
$app->register(Dingo\Api\Provider\LumenServiceProvider::class);
```

### Facades

There are two facades shipped with the package. You can add either of them should you wish.

这个包提供了两个 facades。你可以随意添加任何一个

#### `Dingo\Api\Facade\API`

This is a facade for the dispatcher, however, it also provides helper methods for other methods throughout the package.

这是一个用于api调度的 facade，当然，它也为这个包的其他方法提供辅助方法。

#### `Dingo\Api\Facade\Route`

This is a facade for the API router and can be used to fetch the current route, request, check the current route name, etc.

这是一个用于 API 路由的facade，可以被用于获取当前路由，请求，检查当前路由名称等。

[Configuration-配置 →](https://github.com/liyu001989/dingo-api-wiki-zh/Configuration)
