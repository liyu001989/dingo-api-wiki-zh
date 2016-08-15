An endpoint is simply another term for a route. When talking about APIs many people refer to the routes you visit as an endpoint.

#### Version Groups

To avoid complications with your main application routes this package utilizes its own router. As such we must first get an instance of the API router to create our endpoints.

```php
$api = app('Dingo\Api\Routing\Router');
```

We must now define a version group. This allows us to create the same endpoint for multiple versions should we need to change things down the track.

```php
$api->version('v1', function ($api) {

});
```

If you would like a group to respond to multiple versions you can simply pass an array of versions.

```php
$api->version(['v1', 'v2'], function ($api) {

});
```

You can also treat this group as a standard group for your particular framework by passing an array of attributes as the second parameter.

```php
$api->version('v1', ['middleware' => 'foo'], function ($api) {

});
```

You can also nest regular groups for further customization of some endpoints.

```php
$api->version('v1', function ($api) {
    $api->group(['middleware' => 'foo'], function ($api) {
        // Endpoints registered here will have the "foo" middleware applied.
    });
});
```

#### Creating Endpoints

Once you have a version group you can start to create your endpoints using the `$api` parameter of the group closure.

```php
$api->version('v1', function ($api) {
    $api->get('users/{id}', 'App\Api\Controllers\UserController@show');
});
```

Because endpoints are grouped per version you can use the exact same URI to create a different response for the same endpoint.

```php
$api->version('v1', function ($api) {
    $api->get('users/{id}', 'App\Api\V1\Controllers\UserController@show');
});

$api->version('v2', function ($api) {
    $api->get('users/{id}', 'App\Api\V2\Controllers\UserController@show');
});
```

You can also register resources and controllers using the respective methods.

> Note that you must specify the full namespace to the controller, e.g., `App\Http\Controllers`.

##### Named Routes And Generating URLs

Naming your routes lets you easily generate URLs to them. You can name your routes in the exact same way as you do in Laravel.

```php
$api->get('users/{id}', ['as' => 'users.index', 'uses' => 'Api\V1\UserController@show']);
```

Now you can generate a URL to the named route.

```php
app('Dingo\Api\Routing\UrlGenerator')->version('v1')->route('users.index');
```

You must supply a version so that the URL can be properly generated based on the route within that version. This let's you use the
same name across multiple versions.

#### Viewing Routes In The Console

If you're using Laravel 5.1 you can see the registered routes using Artisan.

```
$ php artisan api:routes
```

This command behaves the same as the `route:list` command that Laravel ships with.

[← Configuration](https://github.com/dingo/api/wiki/Configuration) | [Responses →](https://github.com/dingo/api/wiki/Responses)
