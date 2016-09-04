A big part of this package is being able to perform requests on your API internally. This allows you to build your application on top of a consumable API. An internal request will also return the original data instead of a raw response object which means you get all the syntactical sugar that comes with it.

这个包很大一部分都是可以再内部调用你的 API。这允许你在 API 之上创建你的应用。一个内部请求也会返回原始数据，而不是原始响应对象，这意味着你可以得到所有的语法糖。

To make internal requests we need an instance of our dispatcher.

为了使用内部请求，我们需要一个 dispatcher 的实例。


```php
$dispatcher = app('Dingo\Api\Dispatcher');
```

We can now make internal requests using the standard HTTP verbs.

我们现在可以使用 HTTP 动词创建内部请求。

```php
Route::get('/', function () use ($dispatcher) {
    $users = $dispatcher->get('api/users');

    return View::make('index')->with('users', $users);
});
```

If your controllers use the `Dingo\Api\Routing\Helpers` trait you can use the `$api` property to make internal requests.

如果你的控制器使用 `Dingo\Api\Routing\Helpers` trait，那么你可以使用 `$api` 变量去创建内部请求。

```php
use Dingo\Api\Routing\Helpers;

class HomeController extends Controller
{
    use Helpers;

    public function index()
    {
        $users = $this->api->get('users');

        return view('index')->with('users', $users);
    }
}
```

#### Sending Along Data 发送数据

```php
$dispatcher->with(['name' => 'Jason', 'location' => 'Australia'])->post('users');
```

Or you can add the data as the second parameter to the `post` method (includes other HTTP verbs, not just `post`).

或者你可以添加数据作为 `post` 方法第二个参数（包括其他的 HTTP 动词，不仅仅是 `post`）。

```php
$dispatcher->post('users', ['name' => 'Jason', 'location' => 'Australia']);
```

#### Targeting Specific API Version 指定特定的版本

```php
$dispatcher->version('v2')->get('users');
```

#### Targeting Specific Domain 指定特定的域名

```php
$dispatcher->on('api.example.com')->get('users');
```

#### Attaching Uploaded Files 添加上传文件

There's a number of ways you can attach files to be uploaded. You can pass an array of `Symfony\Component\HttpFoundation\File\UploadedFile` instances, which is handy when sending along files you've just uploaded to the API.

这有很多方法让你添加上传文件。你可以传递一个 `Symfony\Component\HttpFoundation\File\UploadedFile` 实例的数组，当一起发送你刚上传的文件到 API 的时候很有用。

```php
$dispatcher->attach(Input::files())->post('photos');
```

Or you can pass an array of file paths, the key of the array should be the file identifier.

或者你可以传递一个文件路径的数组，数组的 key 是文件的 id。

```php
$dispatcher->attach(['photo' => 'photos/me.jpg'])->post('photos');
```

Or you can pass an array of file paths and associated meta data. Depending on the circumstances this is vastly more accurate then the above method as the package does not need to figure out what the mime type and file size are.

或者你可以传递一个文件路径数据和 meta 数据。根据这个情况，这比上面不关心类型和文件大小的方法更加的准确，

```php
$dispatcher->attach([
    'photo' => [
        'path' => 'photos/me.jpg',
        'mime' => 'image/jpeg',
        'size' => '49430'
    ]
])->post('photos');
```

You can also mix and match the above methods should you need to.

你也可以混合使用上面你需要的方法。

#### Sending JSON Data 发送 JSON 数据

```php
$data = ['name' => 'bill', 'password' => 12345];

$dispatcher->json($data)->post('users');
```

If `$data` is an array it will be automatically encoded. The `Content-Type` of this request will be set to `application/json`.

如果 `$data` 是个数组，他讲被自动的编译。请求头 `Content-Type` 将被设置为 `application/json`。

#### Pretend To Be Authenticated User 伪装为认证用户

If some endpoints on your API require authentication you can internally pretend to be a given user. If, for example, a user is logged in to your application using Laravel's authentication you can retrieve the logged in user and pretend to be that user when performing internal requests.

如果一些 API 需要用户认证，你可以在内部伪装登录一个用户。举个例子，如果一个用户已经使用 Laravel 的 authentication 登录到你的应用，你可以取得登录用户，伪装为那个用户去做内部请求。

```php
$dispatcher->be(auth()->user())->get('posts');
```

Any subsequent requests will be authenticated as the same user. To authenticate as a given user for a single request you can use the `once` method.

任何随后的请求都会认证为相同的用户。如果想为一个请求添加认证用户，你可以使用 `once` 方法。

```php
$dispatcher->be(auth()->user())->once()->get('posts');
```

#### Retrieve Raw Response Object 取回原始响应对象

All internal requests return the pre-transformed and pre-formatted data. If, for example, your API returns an Eloquent collection from an endpoint you will receive that Eloquent collection internally. Should you require the raw response object you just prefix all calls with `raw`.

所有的内部请求会返回预转换和预格式化的手，举个例子，如果你的 API 返回一个 Eloquent 集合，你将在内部收到 Eloquent 集合。如果你需要原始的响应对象，你只要在所有调用前面增加 `raw`。

```php
$response = $dispatcher->raw()->get('users');
```

#### Handling Exceptions 处理异常

Any exceptions thrown in your endpoints will be rethrown for you to manually catch when performing internal requests.

当进行内部请求的时候，你的 API 抛出的任何异常将被重新抛出，方便你手动捕获。

```php
public function store()
{
    throw new Symfony\Component\HttpKernel\Exception\ConflictHttpException('We got a conflict!');
}
```

If we were to internally call the above endpoint we'd need to catch the exception thrown.

如果我们要在内部调用 API，那么我们需要捕获它抛出的异常。

```php
try {
    app('Dingo\Api\Dispatcher')->with($payload)->post('users');
} catch (Symfony\Component\HttpKernel\Exception\ConflictHttpException $e) {
    // Do something here, like return with an error.
}
```

The package may also throw a `Dingo\Api\Exception\InternalHttpException` when an error response is returned. If you're using the response builder to return errors then you'll need to catch the thrown exception. The response will be available on the exception.

当返回一个错误响应时，这个包也会抛出一个 `Dingo\Api\Exception\InternalHttpException` 异常。如果你正在使用响应生成器去返回错误，那么你需要捕获这个异常。响应将可以利用上异常。

```php
public function show($id)
{
    return $this->response->errorNotFound('Could not find the user.');
}
```

When internally calling this endpoint we'll need to catch the exception.

当内部调用这个 API 的时候，我们需要捕获异常。

```php
try {
    app('Dingo\Api\Dispatcher')->get('users/1');
} catch (Dingo\Api\Exception\InternalHttpException $e) {
    // We can get the response here to check the status code of the error or response body.
    $response = $e->getResponse();
}
```

[← Authentication](https://github.com/liyu001989/dingo-api-wiki-zh/blob/master/Authentication.md) | [OAuth 2.0 →](https://github.com/liyu001989/dingo-api-wiki-zh/blob/master/OAuth-2.0.md)
