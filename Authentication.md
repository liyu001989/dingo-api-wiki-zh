Most APIs will usually require some form of authentication prior to returning responses. Sometimes a response may differ when a request is authenticated compared to an unauthenticated request.

This package allows you to configure multiple authentication providers. When authentication is enabled each of the providers is executed in an attempt to authenticate the request.

### Configuring Authentication Providers

By default only HTTP Basic authentication is enabled in the configuration file. Here is a list of the current supported authentication providers that are built in to the package.

- HTTP Basic (`Dingo\Api\Auth\Provider\Basic`)
- JSON Web Tokens (`Dingo\Api\Auth\Provider\JWT`)
- OAuth 2.0 (`Dingo\Api\Auth\Provider\OAuth2`)

#### HTTP Basic

This provider uses the default basic authentication built into Laravel and Lumen. You will need to configure this provider in a service provider or bootstrap file. The second parameter is the identifier used to authenticate.

```php
app('Dingo\Api\Auth\Auth')->extend('basic', function ($app) {
   return new Dingo\Api\Auth\Provider\Basic($app['auth'], 'email');
});
```

#### JSON Web Tokens (JWT)

This package makes use of a 3rd party package to integrate JWT authentication. Please refer to the [`tymon/jwt-auth`](https://github.com/tymondesigns/jwt-auth) GitHub page for details on installing and configuring the package.

Once you have the package you can configure the provider in your `config/api.php` file or in a service provider or bootstrap file.

```php
'jwt' => 'Dingo\Api\Auth\Provider\JWT'
```

```php
app('Dingo\Api\Auth\Auth')->extend('jwt', function ($app) {
   return new Dingo\Api\Auth\Provider\JWT($app['Tymon\JWTAuth\JWTAuth']);
});
```

#### OAuth 2.0

This package makes use of a 3rd party package to integrate OAuth 2.0. You can either install [`league/oauth2-server`](https://github.com/thephpleague/oauth2-server) and configure the server yourself or use the bridge package, [`lucadegasperi/oauth2-server-laravel`](https://github.com/lucadegasperi/oauth2-server-laravel).

> For simplicity this guide will assume you are using the bridge package.

Once you have the package you will need to configure it in a service provider or bootstrap file.

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

##### User And Client Resolvers

Depending on the authorization grants you enable you may not need both of the resolvers. If, for example, you only allow clients to authenticate via OAuth 2.0 then you are not required to set a user resolver.

The resolvers both receive the ID of the user or client and should use this ID to return an instance of the user or client. This usually involves querying the database for the user or client.

### Custom Authentication Providers

If you're developing for a legacy system or require some other form of authentication you may implement your own provider.

Your authentication provider should implement `Dingo\Api\Contract\Auth\Provider`. If authentication succeeds your provider should return an instance of the authenticated user. If authentication fails your provider should thrown a `Symfony\Component\HttpKernel\Exception\UnauthorizedHttpException`.

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

```php
'custom' => 'CustomProvider'
```

Or from your bootstrap file or service provider.

```php
app('Dingo\Api\Auth\Auth')->extend('custom', function ($app) {
    return new CustomProvider;
});
```

### Protecting Endpoints

You can enable or disable protection at the route or group level by enabled the `api.auth` route middleware.

#### Require Authentication On All Routes

```php
$api->version('v1', ['middleware' => 'api.auth'], function ($api) {
    // Routes within this version group will require authentication.
});
```

#### Require Authentication On Specific Routes

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

```php
$api->version('v1', function ($api) {
    $api->get('user', ['middleware' => 'api.auth', 'providers' => ['basic', 'oauth'], function () {
        // This route requires authentication.
    }]);
});
```

#### Require Authentication On Controller Methods

Setting middleware on controllers is available in both Laravel and Lumen. From your constructor you may use the `middleware` method.

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

### Retrieving Authenticated User

Within a protected endpoint you can retrieve the authenticated user.

```php
$api->version('v1', ['middleware' => 'api.auth'], function ($api) {
    $api->get('user', function () {
        $user = app('Dingo\Api\Auth\Auth')->user();

        return $user;
    });
});
```

If your controllers use the `Dingo\Api\Routing\Helpers` trait then you can use the `$auth` property.

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

[← Transformers](https://github.com/dingo/api/wiki/Transformers) | [Rate Limiting →](https://github.com/dingo/api/wiki/Rate-Limiting)
