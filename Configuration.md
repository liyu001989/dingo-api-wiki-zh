Much of the package comes preconfigured so that you can start building your API as soon as possible. You can use your `.env` file to configure most of the package, however, some finer tuning of the package will require you to either publish the configuration file (Laravel) or configure it in `bootstrap/app.php` (Lumen). You may also use the `boot` method of your `AppServiceProvider`.

大部分的包都是预先配置好的，所以你可以尽快创建你的 API。你可以使用 `·env` 文件去配置大部分的包，当然，一些包的微调需要你发布配置文件（laravel）或者在 `bootstrap/app.php` 文件中配置 （lumen）。你也可以使用 `AppServiceProvider` 中的 `boot` 方法。

**Important:** If you're using Laravel 5 you must use either a published configuration file OR a service provider. Do not use the `bootstrap/app.php` file.

**重要：** 如果你正在使用 Laravel 5 你不仅需要一个发布的配置文件，还需要一个 service provider。不要使用 `bootstrap/app.php` 文件。

If you're using Laravel you can publish the configuration file with the following Artisan command:

如果你正在使用 Laravel 你可以使用下面的 Artisan 命令发布配置文件：

```
php artisan vendor:publish --provider="Dingo\Api\Provider\LaravelServiceProvider"
```

#### Standards Tree 标准树

There's three different trees: `x`, `prs`, and `vnd`. The standards tree you use will depend upon the project you're developing.

这有三个不同的树: `x`，`prs` 和 `vnd`。你使用的标准树需要取决于你开发的项目

- The unregistered tree (`x`) is primarily meant for local or private environments.
- 未注册的树（`x`）主要表示本地和私有环境
- The personal tree (`prs`) is primarily meant for projects that are not distributed commerically.
- 私有树（`prs`）主要表示没有商业发布的项目
- The vendor tree (`vnd`) is primarily meant for projects that are publically available and distributed.
- 供应商树（`vnd`）主要表示公开发布的项目

> Subtypes using the personal or vendor trees are *technically* meant to register with the IANA, but are not required to.

> 子类型使用私有和供应商树在**技术上**意味着在 IANA 上注册，但是并不强制要求。

At the end of the day, if you aren't sure which to pick the x or unregistered tree is always a safe bet.

最后，如果你不确定该如何选择, x 树或者说未注册树都是安全的。

You can configure this in your `.env` file.

你可以配置在 `.env` 文件中。

```
API_STANDARDS_TREE=vnd
```

#### Subtype 子类型

Your subtype is typically a short name of your application or project, all lowercase.

你的 subtype 一般是你应用或者项目的简称，全小写。

You can configure this in your `.env` file.

你可以配置在 `.env` 文件中。

```
API_SUBTYPE=myapp
```

#### Prefixes and Subdomains 前缀和子域名

If you've ever worked with an API you'll know that most are served from either a subdomain or under a prefix. A prefix or subdomain *is* required, but only one. Avoid putting a version number as your prefix or subdomain as versioning is handled via the `Accept` header.

如果你曾经写过 API，你应该知道大部分的 API 不是有一个子域名就是有一个前缀。一个前缀或者子域名是必要的，但是只能有一个。应避免将版本号作为前缀或者子域名，版本的变更应该由 `Accept` 头控制。

You can configure this in your `.env` file.

你可以配置在 `.env` 文件中。

```
API_PREFIX=api
```

Or you can use a domain.

或者你可以使用域名

```
API_DOMAIN=api.myapp.com
```

#### Version 版本

This version is the default version of your API and is used as a fallback in several circumstances whenever a version is not supplied. This version is also used
as a default version for when generating API documentation.

version 是你 API 的默认版本, 用于某些状况下没有指定版本号的时候。它也用作生成 API 文档的默认版本。

You can also configure this in your `.env` file.

你可以配置在 `.env` 文件中。

```
API_VERSION=v1
```

#### Name 名称

The name of your API is only used when you're generating documentation using the API Blueprint command. This name is used as a default
to avoid having to manually define the name whenever you generate documentation.

name 只用在你生成文档的时候，当你生成文档的时候用作默认的名字，避免必须去手动定义。

You can also configure this in your `.env` file.

你可以配置在 `.env` 文件中。

```
API_NAME=My API
```

You may need to wrap the name in quotes.

你可能需要用引号包裹这个名字。

```
API_NAME="My API"
```

#### Conditional Requests 条件请求

By default conditional requests are enabled as it will utilize the clients caching capabilities when possible to cache API requests.

默认情况下，有条件的请求默认开启，它将利用客户端的缓存机制在可能的情况下缓存 API 请求。

You can configure this in your `.env` file.

你可以在你的 `.env` 文件中配置。

```
API_CONDITIONAL_REQUEST=false
```

#### Strict Mode 严格模式

Strict mode will require clients to send the `Accept` header instead of defaulting to the version specified in the configuration file. This means you will not be able to browse the API through your web browser.

严格模式需要客户端发送 `Accept` 头，代替配置文件中配置的默认版本。这意味着你将不能通过浏览器访问你的 API。

If strict mode is enabled and an invalid `Accept` header is used the API will throw an unhandled `Symfony\Component\HttpKernel\Exception\BadRequestHttpException` that should be you should handle appropriately.

如果开启严格模式，发送非法的 `Acceept` 会抛出一个未处理的异常 `Symfony\Component\HttpKernel\Exception\BadRequestHttpException` ，你需要自己处理这个异常。 

You can configure this in your `.env` file.

你可以在你的 `.env` 文件中配置。

```
API_STRICT=false
```

#### Authentication Providers

By default only `basic` authentication is enabled. Authentication is covered in more detail in a later chapter.

默认的只有 `basic` 认证是开启的。在后面的章节中会有详细的介绍。

For more complex configuration you will need a service provider or bootstrap file.

更多复杂的配置你需要配置在一个 service provider 或者启动文件中。

```php
$app['Dingo\Api\Auth\Auth']->extend('oauth', function ($app) {
   return new Dingo\Api\Auth\Provider\JWT($app['Tymon\JWTAuth\JWTAuth']);
});
```

#### Throttling / Rate Limiting

By default rate limiting is disabled. You can register your custom throttles with the rate limiter or use the existing authenticated and unauthenticated throttles.

默认情况下 rate limiting 是关闭的。你可以注册你自定义的阀门，或者使用已经存在的认证的和非认证的阀门。

For more complex configuration you will need a service provider or bootstrap file.

更多复杂的配置你需要配置在一个 service provider 或者启动文件中。

```php
$app['Dingo\Api\Http\RateLimit\Handler']->extend(function ($app) {
    return new Dingo\Api\Http\RateLimit\Throttle\Authenticated;
});
```

#### Response Transformer 响应transformer

Fractal is the default response transformer.

Fractal 是默认的响应 transformer。

You can configure this in your `.env`. file, however, for more complex configuration you will need a service provider or bootstrap file.

你可以配置在你的 `.env` 文件中。当然，更加复杂的配置你需要配置在一个 service provider 中或者启动文件中。

```php
$app['Dingo\Api\Transformer\Factory']->setAdapter(function ($app) {
    $fractal = new League\Fractal\Manager;

    $fractal->setSerializer(new League\Fractal\Serializer\JsonApiSerializer);

    return new Dingo\Api\Transformer\Adapter\Fractal($fractal);
});
```

#### Response Formats 格式化响应

The default response format is JSON and a JSON response format is registered by default.

默认的响应会格式化为 JSON，JSON 响应格式是被默认注册的。

You can configure the default response format in your `.env` file. Further response format configuration will need to take place in a published configuration file, a service provider, or in your bootstrap file.

你可以在你的 `.env` 文件中配置你的默认响应格式。更多的响应格式化配置需要配置在一个发布的配置文件、一个 service provider 或者你的启动文件中。

```
API_DEFAULT_FORMAT=json
```

```php
Dingo\Api\Http\Response::addFormatter('json', new Dingo\Api\Http\Response\Format\Jsonp);
```

#### Error Format 格式化错误

When the package encounters errors it will try to generate a generic error response instead of dumping exceptions to the user. The error format it uses can be configured to your liking.

当遇到错误的时候，这个包会尝试生成一个通用的错误响应，而不是将异常打印给用户。你可以在你喜欢的地方配置错误格式化。

You must configure this in a published configuration file or in your bootstrap file.

你必须在一个已发布的配置文件或者启动文件中配置这个。

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

#### Debug Mode 调试模式

Generic errors handled by the package include a `debug` key that will be populated with stack trace details when this is enabled.

当调试模式开启时，一般的错误被包捕获，结果中包含了一个 `debug` 键，值为详细的错误追踪

You can configure this in your `.env` file.

你可以在你的 `.env` 文件中配置。

```
API_DEBUG=true
```

[← Installation](https://github.com/liyu001989/dingo-api-wiki-zh/blob/master/Installation.md) | [Creating API Endpoints →](https://github.com/liyu001989/dingo-api-wiki-zh/blob/master/Creating-API-Endpoints.md)
