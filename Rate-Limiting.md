Rate Limiting (throttling) allows you to limit the number of requests a client can make in a given amount of time. A limit and the expiration time is defined by a throttle. By default the package has two throttles, an authenticated throttle and an unauthenticated throttle.

速率限定（节流）允许你限定客户端每分钟请求的次数。限制和过期时间被阀门决定。默认的，这个包有两种阀门，一个身份验证阀门，一个非身份验证阀门。

### Changing Rate Limiting Key 改变速率限定 key

By default rate limiting is applied to a clients IP address. To change this default behaviour you can register your own resolver which should return a string to be used by the rate limiter.

默认的速率限定被应用到客户端的 IP 地址。要改变默认的行为，你可以注册你自己的解析器，它需要返回一个字符串，供速率限制器使用。

```php
app('Dingo\Api\Http\RateLimit\Handler')->setRateLimiter(function ($app, $request) {
    return $app['example']->getRateLimiterKey();
});
```

The first parameter will be the application IoC container instance and the second is the request instance that is being rate limited.

第一个参数是应用的 IoC 容器实例，第二个参数是被限制的请求实例。

### Enabling Rate Limiting 开启速率限定

To enable rate limiting for a route or group of routes you must enable the `api.throttle` middleware. Once rate limiting is enabled you must have configured some throttles or configured route specific throttling.

要为一个路由或一组路由开启速率限定，你需要使用 `api.throttle` 中间件。一旦速率限定开启了，你需要配置一些阀门或者配置路由的具体限制。

#### Require Rate Limiting On All Routes 所有路由需要速率限定

```php
$api->version('v1', ['middleware' => 'api.throttle'], function ($api) {
    // Routes within this version group will require authentication.
});
```

#### Require Rate Limiting On Specific Routes 特殊的路由进行速率限定

```php
$api->version('v1', function ($api) {
    $api->get('user', ['middleware' => 'api.throttle', function () {
        // This route requires authentication.
    }]);

    $api->get('posts', function () {
        // This route does not require authentication.
    });
});
```

### Route Specific Throttling 路由特殊限制

If you want to only rate limit certain routes or groups of routes you can use the `limit` and `expires` options in your routes.

如果你只想限制某些路由或分组路由，你可以使用 `limit` 和 `expires` 参数到你的路由。

```php
$api->version('v1', function ($api) {
    $api->get('users', ['middleware' => 'api.throttle', 'limit' => 100, 'expires' => 5, function () {
        return User::all();
    }]);
});
```

This would set a request limit of 100 with an expiration time of 5 minutes for this specific route. If you were to set it on the group then each route within the group would have a limit of 100.

这个路由将被设置为 100 次请求限制和 5 分钟的过期时间。如果你将这个设置在分组上，那么分组中的所有路由都有 100 次的限制。

```php
$api->version('v1', ['middleware' => 'api.throttle', 'limit' => 100, 'expires' => 5], function ($api) {
    $api->get('users', function () {
        return User::all();
    });

    $api->get('posts', function () {
        return Post::all();
    });
});
```

A user could visit both the `/users` route and the `/posts` route 100 times each. The limit does not apply to the entire group but to each route within the group.

一个用户可以访问每个 `/users` 路由和 `/posts` 路由 100 次。这个限制不会作用于所有的分组，只会作用都这个个分组中的每个路由。

### Custom Throttles 自定义阀门

You may need a custom throttle for more complex scenarios where you need to meet a couple of conditions in order for the throttle to be applied. A throttle must implement the `Dingo\Api\Contract\Http\RateLimit\Throttle`, however, an abstract class does exists to quickly get started. Each of the predefined throttles extends this abstract class.

你可能需要为更复杂的情况自定义阀门，为了实现，你需要满足一些条件。一个阀门必须实现 `Dingo\Api\Contract\Http\RateLimit\Throttle` 接口，然而，一个抽象类可以帮助我们快速上手。每一个预定义阀门继承这个抽象类。

All a throttle does is attempt to match a given condition. The throttle should return `true` or `false` depending on whether or not it matches the condition. As an example you might want to match that an authenticated user belongs to a given group.、

所有的阀门都在尝试匹配给定的情况。阀门需要根据他是否匹配条件返回 `true` 或 `false`。举个例子，你可能想要匹配一个指定组的认证的用户。

```php
use Illuminate\Container\Container;
use Dingo\Api\Http\RateLimit\Throttle\Throttle;

class CustomThrottle extends Throttle
{
    public function match(Container $app)
    {
        // Perform some logic here and return either true or false depending on whether
        // your conditions matched for the throttle.
    }
}
```

You can then configure your throttle.

接下来，你可以配置你的阀门。

```php
'throttling' => [
    'custom' => new CustomThrottle(['limit' => 200, 'expires' => 10])
]
```

Or register it in your Lumen bootstrap file.

或者再你的 Lumen 启动文件中注册它。

```php
app('Dingo\Api\Http\RateLimit\Handler')->extend(new CustomThrottle(['limit' => 200, 'expires' => 10]));
```

[← Authentication](https://github.com/liyu001989/dingo-api-wiki-zh/blob/master/Authentication.md) | [Internal Requests →](https://github.com/liyu001989/dingo-api-wiki-zh/blob/master/Internal-Requests.md)
