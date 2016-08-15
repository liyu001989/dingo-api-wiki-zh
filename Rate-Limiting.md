Rate Limiting (throttling) allows you to limit the number of requests a client can make in a given amount of time. A limit and the expiration time is defined by a throttle. By default the package has two throttles, an authenticated throttle and an unauthenticated throttle.

### Changing Rate Limiting Key

By default rate limiting is applied to a clients IP address. To change this default behaviour you can register your own resolver which should return a string
to be used by the rate limiter.

```php
app('Dingo\Api\Http\RateLimit\Handler')->setRateLimiter(function ($app, $request) {
    return $app['example']->getRateLimiterKey();
});
```

The first parameter will be the application IoC container instance and the second is the request instance that is being rate limited.

### Enabling Rate Limiting

To enable rate limiting for a route or group of routes you must enable the `api.throttle` middleware. Once rate limiting is enabled you must have configured some throttles or configured route specific throttling.

#### Require Rate Limiting On All Routes

```php
$api->version('v1', ['middleware' => 'api.throttle'], function ($api) {
    // Routes within this version group will require authentication.
});
```

#### Require Rate Limiting On Specific Routes

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

### Route Specific Throttling

If you want to only rate limit certain routes or groups of routes you can use the `limit` and `expires` options in your routes.

```php
$api->version('v1', function ($api) {
    $api->get('users', ['middleware' => 'api.throttle', 'limit' => 100, 'expires' => 5, function () {
        return User::all();
    }]);
});
```

This would set a request limit of 100 with an expiration time of 5 minutes for this specific route. If you were to set it on the group then each route within the group would have a limit of 100.

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

### Custom Throttles

You may need a custom throttle for more complex scenarios where you need to meet a couple of conditions in order for the throttle to be applied. A throttle must implement the `Dingo\Api\Contract\Http\RateLimit\Throttle`, however, an abstract class does exists to quickly get started. Each of the predefined throttles extends this abstract class.

All a throttle does is attempt to match a given condition. The throttle should return `true` or `false` depending on whether or not it matches the condition. As an example you might want to match that an authenticated user belongs to a given group.

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

```php
'throttling' => [
    'custom' => new CustomThrottle(['limit' => 200, 'expires' => 10])
]
```

Or register it in your Lumen bootstrap file.

```php
app('Dingo\Api\Http\RateLimit\Handler')->extend(new CustomThrottle(['limit' => 200, 'expires' => 10]));
```

[← Authentication](https://github.com/dingo/api/wiki/Authentication) | [Internal Requests →](https://github.com/dingo/api/wiki/Internal-Requests)
