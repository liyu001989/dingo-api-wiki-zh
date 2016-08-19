Much of the package comes preconfigured so that you can start building your API as soon as possible. You can use your `.env` file to configure most of the package, however, some finer tuning of the package will require you to either publish the configuration file (Laravel) or configure it in `bootstrap/app.php` (Lumen). You may also use the `boot` method of your `AppServiceProvider`.

大部分的包都是预先配置的，所以你可以尽快创建你的API。你可以使用你的 `·env` 文件去配置大部分的包，当然，一些包的微调需要你发布配置文件（laravel）或者配置在 `bootstrap/app.php` （lumen）。你也可以使用 `AppServiceProvider` 中的 `boot` 方法。

**Important:** If you're using Laravel 5 you must use either a published configuration file OR a service provider. Do not use the `bootstrap/app.php` file.

**重要：** 如果你正在使用 Laravel 5 你不但需要一个发布的配置文件，还需要一个 service provider。不要使用`bootstrap/app.php` 文件。

If you're using Laravel you can publish the configuration file with the following Artisan command:

如果你正在使用 Laravel 你可以使用下面的 Artisan 命令发布配置文件。

```
php artisan vendor:publish --provider="Dingo\Api\Provider\LaravelServiceProvider"
```

#### Standards Tree 标准树

There's three different trees: `x`, `prs`, and `vnd`. The standards tree you use will depend upon the project you're developing.

这有三个不用的树: `x`，`prs` 和 `vnd`。你使用的标准树需要取决于你开发的项目

- The unregistered tree (`x`) is primarily meant for local or private environments.
- 未登记的树（`x`）主要表示本地和私有环境
- The personal tree (`prs`) is primarily meant for projects that are not distributed commerically.
- 私有树（`prs`）主要表示没有商业发布的项目
- The vendor tree (`vnd`) is primarily meant for projects that are publically available and distributed.
- 供应商树（`vnd`）主要表示公开发布的项目

> Subtypes using the personal or vendor trees are *technically* meant to register with the IANA.

> 子类型使用私有和供应商树在**技术上**意味着在 IANA 上注册。

You can configure this in your `.env` file.

你可以再你的 `.env` 文件中配置这些

```
API_STANDARDS_TREE=vnd
```

#### Subtype 子类型 

Your subtype is typically a short name of your application or project, all lowercase.

你的 subtype 一般是你的应用或者项目的简称，全小写。

You can configure this in your `.env` file.

你可以在你的 `.env` 文件中配置。

```
API_SUBTYPE=myapp
```

#### Prefixes and Subdomains 前缀和子域名 

If you've ever worked with an API you'll know that most are served from either a subdomain or under a prefix. A prefix or subdomain *is* required, but only one. Avoid putting a version number as your prefix or subdomain as versioning is handled via the `Accept` header.

You can configure this in your `.env` file.

```
API_PREFIX=api
```

Or you can use a domain.

或者你可以使用域名

```
API_DOMAIN=api.myapp.com
```

#### Version

This version is the default version of your API and is used as a fallback in several circumstances whenever a version is not supplied. This version is also used
as a default version for when generating API documentation.

You can also configure this in your `.env` file.

```
API_VERSION=v1
```

#### Name

The name of your API is only used when you're generating documentation using the API Blueprint command. This name is used as a default
to avoid having to manually define the name whenever you generate documentation.

You can also configure this in your `.env` file.

```
API_NAME=My API
```

You may need to wrap the name in quotes.

```
API_NAME="My API"
```

#### Conditional Requests

By default conditional requests are enabled as it will utilize the clients caching capabilities when possible to cache API requests.

You can configure this in your `.env` file.

```
API_CONDITIONAL_REQUEST=false
```

#### Strict Mode

Strict mode will require clients to send the `Accept` header instead of defaulting to the version specified in the configuration file. This means you will not be able to browse the API through your web browser.

If strict mode is enabled and an invalid `Accept` header is used the API will throw an unhandled `Symfony\Component\HttpKernel\Exception\BadRequestHttpException` that should be you should handle appropriately.

You can configure this in your `.env` file.

```
API_STRICT=false
```

#### Authentication Providers

By default only `basic` authentication is enabled. Authentication is covered in more detail in a later chapter.

For more complex configuration you will need a service provider or bootstrap file.

```php
$app['Dingo\Api\Auth\Auth']->extend('oauth', function ($app) {
   return new Dingo\Api\Auth\Provider\JWT($app['Tymon\JWTAuth\JWTAuth']);
});
```

#### Throttling / Rate Limiting

By default rate limiting is disabled. You can register your custom throttles with the rate limiter or use the existing authenticated and unauthenticated throttles.

For more complex configuration you will need a service provider or bootstrap file.

```php
$app['Dingo\Api\Http\RateLimit\Handler']->extend(function ($app) {
    return new Dingo\Api\Http\RateLimit\Throttle\Authenticated;
});
```

#### Response Transformer

Fractal is the default response transformer.

You can configure this in your `.env`. file, however, for more complex configuration you will need a service provider or bootstrap file.

```php
$app['Dingo\Api\Transformer\Factory']->setAdapter(function ($app) {
    $fractal = new League\Fractal\Manager;

    $fractal->setSerializer(new League\Fractal\Serializer\JsonApiSerializer);

    return new Dingo\Api\Transformer\Adapter\Fractal($fractal);
});
```

#### Response Formats

The default response format is JSON and a JSON response format is registered by default.

You can configure the default response format in your `.env` file. Further response format configuration will need to take place in a published configuration file, a service provider, or in your bootstrap file.

```
API_DEFAULT_FORMAT=json
```

```php
Dingo\Api\Http\Response::addFormatter('json', new Dingo\Api\Http\Response\Format\Jsonp);
```

#### Error Format

When the package encounters errors it will try to generate a generic error response instead of dumping exceptions to the user. The error format it uses can be configured to your liking.

You must configure this in a published configuration file or in your bootstrap file.

```php
$app['Dingo\Api\Exception\Handler']->setErrorFormat([
    'error' => [
        'message' => ':message',
        'errors' => ':errors',
        'code' => ':code',
        'status_code' => ':status_code',
        'debug' => ':debug'
    ]
]);
```

#### Debug Mode

Generic errors handled by the package include a `debug` key that will be populated with stack trace details when this is enabled.

You can configure this in your `.env` file.

```
API_DEBUG=true
```

[← Installation](https://github.com/liyu001989/dingo-api-wiki-zh/blob/master/Installation.md) | [Creating API Endpoints →](https://github.com/liyu001989/dingo-api-wiki-zh/blob/master/Creating-API-Endpoints.md)
