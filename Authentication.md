Most APIs will usually require some form of authentication prior to returning responses. Sometimes a response may differ when a request is authenticated compared to an unauthenticated request.

许多 API 通常在返回响应之前都需要某种形式的认证，有些时候，一个认证的请求和一个未认证的请求，响应可能不同。

This package allows you to configure multiple authentication providers. When authentication is enabled each of the providers is executed in an attempt to authenticate the request.

dingo/api 允许你配置不同的认证 providers。当开启认证，每个 providers 都会在尝试认证请求的时候被执行。

### Configuring Authentication Providers

By default only HTTP Basic authentication is enabled in the configuration file. Here is a list of the current supported authentication providers that are built in to the package.

默认的，配置文件中只开启了 基础的 HTTP 认证。下面是目前内置支持的认证 providers。

- HTTP Basic (`Dingo\Api\Auth\Provider\Basic`)
- JSON Web Tokens (`Dingo\Api\Auth\Provider\JWT`)
- OAuth 2.0 (`Dingo\Api\Auth\Provider\OAuth2`)

#### HTTP Basic

This provider uses the default basic authentication built into Laravel and Lumen. You will need to configure this provider in a service provider or bootstrap file. The second parameter is the identifier used to authenticate.

Laravel 和 Lumen 内置默认使用基础认证。你需要再一个 service provider 或启动文件中配置它。第二个参数是用于认证的 id。

```php
app('Dingo\Api\Auth\Auth')->extend('basic', function ($app) {
   return new Dingo\Api\Auth\Provider\Basic($app['auth'], 'email');
});
```

#### JSON Web Tokens (JWT)

This package makes use of a 3rd party package to integrate JWT authentication. Please refer to the [`tymon/jwt-auth`](https://github.com/tymondesigns/jwt-auth) GitHub page for details on installing and configuring the package.

dingo/api 使用第三方包集成 JWT 认证。请参阅 [`tymon/jwt-auth`](https://github.com/tymondesigns/jwt-auth) Github page，查看安装和配置的详细信息。

Once you have the package you can configure the provider in your `config/api.php` file or in a service provider or bootstrap file.

一旦你安装成功，你可以在你的 `config/api.php` 文件或者一个 service provider 或者启动文件中配置它。

```php
'auth' => [
    'jwt' => 'Dingo\Api\Auth\Provider\JWT',
],
```

```php
app('Dingo\Api\Auth\Auth')->extend('jwt', function ($app) {
   return new Dingo\Api\Auth\Provider\JWT($app['Tymon\JWTAuth\JWTAuth']);
});
```

#### OAuth 2.0

This package makes use of a 3rd party package to integrate OAuth 2.0. You can either install [`league/oauth2-server`](https://github.com/thephpleague/oauth2-server) and configure the server yourself or install and configure the bridge package, [`lucadegasperi/oauth2-server-laravel`](https://github.com/lucadegasperi/oauth2-server-laravel).

dingo/api 使用第三方包集成 OAuth 2.0。你可以安装[`league/oauth2-server`](https://github.com/thephpleague/oauth2-server) 然后自己配置服务端，或者安装和配置集成好的包，[`lucadegasperi/oauth2-server-laravel`](https://github.com/lucadegasperi/oauth2-server-laravel).

> For simplicity this guide will assume you are using the bridge package.
> 为了简单期间，这个教程将假设你使用了集成好的包。

Once you have the package you will need to configure it in a service provider or bootstrap file.

一旦你安装了这个包，你可以在 service provider 或者启动文件中配置它。

```php
app('Dingo\Api\Auth\Auth')->extend('oauth', function ($app) {
   $provider = new Dingo\Api\Auth\Provider\OAuth2($app['oauth2-server.authorizer']->getChecker());

    $provider->setUserResolver(function ($id) {
        // Logic to return a user by their ID.
    });

    $provider->setClientResolver(function ($id) {
        // Logic to return a client by their ID.
    });

    return $provider;
});
```

Or from within a service provider.

或者从一个 service provider 中配置

```php
namespace App\Providers;

use Dingo\Api\Auth\Auth;
use Dingo\Api\Auth\Provider\OAuth2;
use Illuminate\Support\ServiceProvider;

class OAuthServiceProvider extends ServiceProvider
{
    public function boot()
    {
        $this->app[Auth::class]->extend('oauth', function ($app) {
            $provider = new OAuth2($app['oauth2-server.authorizer']->getChecker());

            $provider->setUserResolver(function ($id) {
                // Logic to return a user by their ID.
            });

            $provider->setClientResolver(function ($id) {
                // Logic to return a client by their ID.
            });

            return $provider;
        });
    }

    public function register()
    {
        //
    }
}
```

##### User And Client Resolvers 用户和客户端解释器

Depending on the authorization grants you enable you may not need both of the resolvers. If, for example, you only allow clients to authenticate via OAuth 2.0 then you are not required to set a user resolver.

根据你启用的授权，你可能不需要这两个解释器。举个例子，你仅仅允许客户端通过 OAuth 2.0 认证，那么你不需要设置用户解释器。

The resolvers both receive the ID of the user or client and should use this ID to return an instance of the user or client. This usually involves querying the database for the user or client.

两个解释器都接受用户和客户端的 ID，应该用这个 ID 返回用户或者客户端的实例。这通常涉及在数据库中查询用户或者客户端。

### Custom Authentication Providers 自定义认证客户端

If you're developing for a legacy system or require some other form of authentication you may implement your own provider.

如果你正在开发一个历史遗留的系统或者需要一些其他的认证方式，你可以实现你自己的 provider。

Your authentication provider should implement `Dingo\Api\Contract\Auth\Provider`. If authentication succeeds your provider should return an instance of the authenticated user. If authentication fails your provider should thrown a `Symfony\Component\HttpKernel\Exception\UnauthorizedHttpException`.

你的认证 provider 需要实现 `Dingo\Api\Contract\Auth\Provider` 接口。如果认证成功，你的provider需要返回一个认证的用户的实例。如果验证失败，你的 provider 需要抛出一个 `Symfony\Component\HttpKernel\Exception\UnauthorizedHttpException` 异常。

```php
use Illuminate\Http\Request;
use Dingo\Api\Routing\Route;
use Dingo\Api\Contract\Auth\Provider;
use Symfony\Component\HttpKernel\Exception\UnauthorizedHttpException;

class CustomProvider implements Provider
{
    public function authenticate(Request $request, Route $route)
    {
        // Logic to authenticate the request.

        throw new UnauthorizedHttpException('Unable to authenticate with supplied username and password.');
    }
}
```

The abstract `Dingo\Api\Auth\Provider\Authorization` can be extended should your provider utilize tokens sent via the `Authorization` header. The `Dingo\Api\Auth\Provider\Authorization::validateAuthorizationHeader` method allows you to easily validate that the authorization header exists and contains a valid value.

抽象类 `Dingo\Api\Auth\Provider\Authorization` 可以被继承，你的 provider 应该利用随着 `Authorization` 头发送过来的 tokens。`Dingo\Api\Auth\Provider\Authorization::validateAuthorizationHeader`
 方法允许你轻松的验证认证头是否存在而且包含一个有效值。

```php

use Illuminate\Http\Request;
use Dingo\Api\Routing\Route;
use Dingo\Api\Auth\Provider\Authorization;
use Symfony\Component\HttpKernel\Exception\BadRequestHttpException;
use Symfony\Component\HttpKernel\Exception\UnauthorizedHttpException;

class CustomProvider extends Authorization
{
    public function authenticate(Request $request, Route $route)
    {
        $this->validateAuthorizationHeader($request);

        // If the authorization header passed validation we can continue to authenticate.
        // If authentication then fails we must throw the UnauthorizedHttpException.
    }

    public function getAuthorizationMethod()
    {
        return 'mac';
    }
}
```

Once you've implemented your authentication provider you can configure it in your `config/api.php` file.

一旦你实现了你的认证 provider，你可以在你的 `config/api.php` 文件中配置它。

```php
'auth' => [
    'custom' => 'CustomProvider',
],
```

Or from your bootstrap file or service provider.

或者从你的启动文件或者 service provider

```php
app('Dingo\Api\Auth\Auth')->extend('custom', function ($app) {
    return new CustomProvider;
});
```

### Protecting Endpoints

You can enable or disable protection at the route or group level by enabled the `api.auth` route middleware.

你可以开启或关闭路由保护在一个或一组路由上，通过使用 `api.auth` 路由中间件。

> If you are using OAuth2, and using the bridge package, you do not need to register it's middlewares. The `api.auth` middleware handles everything, but doesn't hurt to register them anyway.

#### Require Authentication On All Routes 所有路由需要认证

```php
$api->version('v1', ['middleware' => 'api.auth'], function ($api) {
    // Routes within this version group will require authentication.
});
```

#### Require Authentication On Specific Routes 特定路由需要认证

```php
$api->version('v1', function ($api) {
    $api->get('user', ['middleware' => 'api.auth', function () {
        // This route requires authentication.
    }]);

    $api->get('posts', function () {
        // This route does not require authentication.
    });
});
```

#### Allow Only Specific Authentication Providers

If you want to set a specific authentication provider on a group of routes or specific route you can do so using the `providers` key.

如果你想在一组路由或特定路由上设置认证的 provider，你可以使用 `providers` 键.

```php
$api->version('v1', function ($api) {
    $api->get('user', ['middleware' => 'api.auth', 'providers' => ['basic', 'oauth'], function () {
        // This route requires authentication.
    }]);
});
```

#### Require Authentication On Controller Methods 控制器方法需要认证

Setting middleware on controllers is available in both Laravel and Lumen. From your constructor you may use the `middleware` method.

Laravel 和 Lumen 中都可以在控制器中设置中间件。再你的构造方法中你可以使用 `middleware` 方法.

```php
class UserController extends Illuminate\Routing\Controller
{
    use Helpers;

    public function __construct()
    {
        $this->middleware('api.auth');

        // Only apply to a subset of methods.
        $this->middleware('api.auth', ['only' => ['index']]);
    }

    public function index()
    {
        //
    }

    public function posts()
    {
        //
    }
}
```

### Retrieving Authenticated User 取得认证的用户

Within a protected endpoint you can retrieve the authenticated user.

在一个受保护的 API 中，你可以得到认证的用户。

```php
$api->version('v1', ['middleware' => 'api.auth'], function ($api) {
    $api->get('user', function () {
        $user = app('Dingo\Api\Auth\Auth')->user();

        return $user;
    });
});
```

If your controllers use the `Dingo\Api\Routing\Helpers` trait then you can use the `$auth` property.

如果你的控制器使用了 `Dingo\Api\Routing\Helpers` trait，那么你可以使用 `$auth` 属性。

```php
use Dingo\Api\Routing\Helpers;
use Illuminate\Routing\Controller;

class UserController extends Controller
{
    use Helpers;

    public function __construct()
    {
        $this->middleware('api.auth');
    }

    public function index()
    {
        $user = $this->auth->user();

        return $user;
    }
}
```

### Optional Authentication

Sometimes you may need to adjust a response based on whether or not the request was authenticated. To do this the route should not be protected. You then simply ask for the authenticated user.

有时候你可能需要给予请求是否认证调整响应。要这样做，那么路由不应该被保护。然后你可以简单的询问认证的用户。

```php
$api->version('v1', function ($api) {
    $api->get('users/{id}', function ($id) {
        $user = User::findOrFail($id);

        // Attempt to authenticate the request. If the request is not authenticated
        // then we'll hide the e-mail from the response. Only authenticated
        // requests can see other users e-mails.
        if (! app('Dingo\Api\Auth\Auth')->user()) {
            $hidden = $user->getHidden();

            $user->setHidden(array_merge($hidden, ['email']));
        }

        return $user;
    });
});
```

[← Transformers](https://github.com/liyu001989/dingo-api-wiki-zh/blob/master/Transformers.md) | [Rate Limiting →](https://github.com/liyu001989/dingo-api-wiki-zh/blob/master/Rate-Limiting.md)
