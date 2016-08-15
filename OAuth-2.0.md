See the [Authentication](https://github.com/dingo/api/wiki/Authentication) chapter for a guide on how to configure an OAuth 2.0 provider.

### Defining Route Scopes

By using scopes you'll have more control over who can access your protected endpoints. Scopes can be set on a group or a route as either an array of pipe delimited string.

#### Route Group Scopes

```php
$api->version('v1', ['middleware' => 'api.auth', 'scopes' => ['read_user_data', 'write_user_data']], function ($api) {
    // Only access tokens with the "read_user_data" scope will be given access.
});
```

#### Specific Route Scopes

```php
$api->version('v1', ['middleware' => 'api.auth'], function ($api) {
    $api->get('user', ['scopes' => 'read_user_data', function () {
        // Only access tokens with the "read_user_data" scope will be given access.
    }]);
});
```

#### Controller Scopes

If your controllers use the `Dingo\Api\Routing\Helpers` trait you can use the `scopes` method.

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

[← Internal Requests](https://github.com/dingo/api/wiki/Internal-Requests) | [Making Requests To Your API →](https://github.com/dingo/api/wiki/Making-Requests-To-Your-API)
