An endpoint is simply another term for a route. When talking about APIs many people refer to the routes you visit as an endpoint.

一个端点是一个路由的另一种说法。当讨论 API 的时候，很多人把访问的路由称作一个端点。

#### Version Groups 版本组

To avoid complications with your main application routes this package utilizes its own router. As such we must first get an instance of the API router to create our endpoints.

为了避免与你主要的项目路由冲突，dingo/api 利用了他自己的路由。要创建端点，我们首先需要获得一个 API 路由的实例。

```php
$api = app('Dingo\Api\Routing\Router');
```

We must now define a version group. This allows us to create the same endpoint for multiple versions should we need to change things down the track.

现在我们必须定义一个版本分组。这允许我们为多个版本创建相同的端点。

```php
$api->version('v1', function ($api) {

});
```

If you would like a group to respond to multiple versions you can simply pass an array of versions.

如果你想一个分组返回多个版本，只需要传递一个版本数组。

```php
$api->version(['v1', 'v2'], function ($api) {

});
```

You can also treat this group as a standard group for your particular framework by passing an array of attributes as the second parameter.

通过在第二个参数上传递一个属性数组，你可以把这个分组当做你自己特殊框架的标准分组。

```php
$api->version('v1', ['middleware' => 'foo'], function ($api) {

});
```

You can also nest regular groups for further customization of some endpoints.

你也可以为某些特殊的端点嵌套使用分组。

```php
$api->version('v1', function ($api) {
    $api->group(['middleware' => 'foo'], function ($api) {
        // Endpoints registered here will have the "foo" middleware applied.
    });
});
```

#### Creating Endpoints 创建端点

Once you have a version group you can start to create your endpoints using the `$api` parameter of the group closure.

一旦你有了一个版本分组，你就可以在分组闭包的参数中，通过 `$api` 创建端点。

```php
$api->version('v1', function ($api) {
    $api->get('users/{id}', 'App\Api\Controllers\UserController@show');
});
```

Because endpoints are grouped per version you can use the exact same URI to create a different response for the same endpoint.

因为端点被每个版本分组了，你可以为相同 URL 上的同一个端点创建不同响应。

```php
$api->version('v1', function ($api) {
    $api->get('users/{id}', 'App\Api\V1\Controllers\UserController@show');
});

$api->version('v2', function ($api) {
    $api->get('users/{id}', 'App\Api\V2\Controllers\UserController@show');
});
```

You can also register resources and controllers using the respective methods.

你也可以使用各自的方法注册资源和控制器。

> Note that you must specify the full namespace to the controller, e.g., `App\Http\Controllers`.

> 提醒，你需要为控制器添加说明完整的命名空间，比如，`App\Http\Controllers`。

##### Named Routes And Generating URLs 命名路由和生成 URLs

Naming your routes lets you easily generate URLs to them. You can name your routes in the exact same way as you do in Laravel.

命名你的路由可以使你方便的生成他们的 URL。你可以跟 Laravel 一样的方法命名你的路由。

```php
$api->get('users/{id}', ['as' => 'users.index', 'uses' => 'Api\V1\UserController@show']);
```

Now you can generate a URL to the named route.

现在你可以为命名了的路由生成 URL 了。

```php
app('Dingo\Api\Routing\UrlGenerator')->version('v1')->route('users.index');
```

You must supply a version so that the URL can be properly generated based on the route within that version. This let's you use the
same name across multiple versions.

你需要提供一个版本，这样才能基于这个版本的路由生成正确的 URL。这允许你在不同版本中使用相同的名字。

#### Viewing Routes In The Console 再命令行查看路由

If you're using Laravel 5.1 you can see the registered routes using Artisan.

如果你正在使用 Laravel 5.1 你可以使用 Artisan 命令看到你注册的路由。

```
$ php artisan api:routes
```

This command behaves the same as the `route:list` command that Laravel ships with.

这个命令和 Laravel 自带的 `route:list` 命令表现相同。

[← Configuration](https://github.com/liyu001989/dingo-api-wiki-zh/blob/master/Configuration.md) | [Responses →](https://github.com/liyu001989/dingo-api-wiki-zh/blob/master/Responses.md)
