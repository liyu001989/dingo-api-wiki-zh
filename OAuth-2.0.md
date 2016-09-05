See the [Authentication](https://github.com/liyu001989/dingo-api-wiki-zh/blob/master/Authentication.md) chapter for a guide on how to configure an OAuth 2.0 provider.

见 [Authentication - 认证](https://github.com/liyu001989/dingo-api-wiki-zh/blob/master/Authentication.md) 章节，了解怎么配置 OAuth 2.0 provider。

### Defining Route Scopes 定义路由 scopes

By using scopes you'll have more control over who can access your protected endpoints. Scopes can be set on a group or a route as either an array of pipe delimited string.

通过使用 scopes，你可以对你的受保护的 API 拥有更多的控制。scopes 可以被设置在一个分组或一个路由上，使用数组或者管道分隔的字符串。

#### Route Group Scopes 路由分组 scopes

```php
$api->version('v1', ['middleware' => 'api.auth', 'scopes' => ['read_user_data', 'write_user_data']], function ($api) {
    // Only access tokens with the "read_user_data" scope will be given access.
});
```

#### Specific Route Scopes 特定路由 scopes

```php
$api->version('v1', ['middleware' => 'api.auth'], function ($api) {
    $api->get('user', ['scopes' => 'read_user_data', function () {
        // Only access tokens with the "read_user_data" scope will be given access.
    }]);
});
```

#### Controller Scopes 控制器 Scopes

If your controllers use the `Dingo\Api\Routing\Helpers` trait you can use the `scopes` method.

如果你的控制器使用 `Dingo\Api\Routing\Helpers`  trait，那么你可以使用 `scopes` 方法。

```php
use Dingo\Api\Routing\Helpers;

class UserController extends Controller
{
    use Helpers;

    public function __construct()
    {
        $this->scopes('read_user_data');
    }
}
```

You can define the methods you want the scopes to apply to via the second parameter, either as a pipe separated string or as an array. If you do not supply the methods then the scopes will apply to all methods. You can also use the `except` and `only` array keys to apply the scopes to a subset of methods.

利用第二个参数，你可以定义 scopes 申请的放阿飞，可以使用管道分隔的字符串或者一个数组。如果你没有提供方法，那么 scopes 将适用于所有方法。你也可以适用 `except`  和  `only` 数组键去适用于方法的子集。

```php
use Dingo\Api\Routing\Helpers;

class UserController extends Controller
{
    use Helpers;

    public function __construct()
    {
        // Only apply to the index method.
        $this->scopes('read_user_data', 'index');

        // Apply to every method except the store method.
        $this->scopes('read_user_data', ['except' => 'store']);

        // Apply only to the store method.
        $this->scopes('write_user_data', ['only' => ['store']]);
    }

    public function index()
    {
        //
    }

    public function store()
    {
        //
    }
}
```

[← Internal Requests](https://github.com/liyu001989/dingo-api-wiki-zh/blob/master/Internal-Requests.md) | [Making Requests To Your API →](https://github.com/liyu001989/dingo-api-wiki-zh/blob/master/Making-Requests-To-Your-API.md)
