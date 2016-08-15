A big part of this package is being able to perform requests on your API internally. This allows you to build your application on top of a consumable API. An internal request will also return the original data instead of a raw response object which means you get all the syntactical sugar that comes with it.

To make internal requests we need an instance of our dispatcher.

```php
$dispatcher = app('Dingo\Api\Dispatcher');
```

We can now make internal requests using the standard HTTP verbs.

```php
Route::get('/', function () use ($dispatcher) {
    $users = $dispatcher->get('api/users');

    return View::make('index')->with('users', $users);
});
```

If your controllers use the `Dingo\Api\Routing\Helpers` trait you can use the `$api` property to make internal requests.

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

#### Sending Along Data

```php
$dispatcher->with(['name' => 'Jason', 'location' => 'Australia'])->post('users');
```

Or you can add the data as the second parameter to the `post` method (includes other HTTP verbs, not just `post`).

```php
$dispatcher->post('users', ['name' => 'Jason', 'location' => 'Australia']);
```

#### Targeting Specific API Version

```php
$dispatcher->version('v2')->get('users');
```

#### Targeting Specific Domain

```php
$dispatcher->on('api.example.com')->get('users');
```

#### Attaching Uploaded Files

There's a number of ways you can attach files to be uploaded. You can pass an array of `Symfony\Component\HttpFoundation\File\UploadedFile` instances, which is handy when sending along files you've just uploaded to the API.

```php
$dispatcher->attach(Input::files())->post('photos');
```

Or you can pass an array of file paths, the key of the array should be the file identifier.

```php
$dispatcher->attach(['photo' => 'photos/me.jpg'])->post('photos');
```

Or you can pass an array of file paths and associated meta data. Depending on the circumstances this is vastly more accurate then the above method as the package does not need to figure out what the mime type and file size are.

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

#### Sending JSON Data

```php
$data = ['name' => 'bill', 'password' => 12345];

$dispatcher->json($data)->post('users');
```

If `$data` is an array it will be automatically encoded. The `Content-Type` of this request will be set to `application/json`.

#### Pretend To Be Authenticated User

If some endpoints on your API require authentication you can internally pretend to be a given user. If, for example, a user is logged in to your application using Laravel's authentication you can retrieve the logged in user and pretend to be that user when performing internal requests.

```php
$dispatcher->be(auth()->user())->get('posts');
```

Any subsequent requests will be authenticated as the same user. To authenticate as a given user for a single request you can use the `once` method.

```php
$dispatcher->be(auth()->user())->once()->get('posts');
```

#### Retrieve Raw Response Object

All internal requests return the pre-transformed and pre-formatted data. If, for example, your API returns an Eloquent collection from an endpoint you will receive that Eloquent collection internally. Should you require the raw response object you just prefix all calls with `raw`.

```php
$response = $dispatcher->raw()->get('users');
```

#### Handling Exceptions

Any exceptions thrown in your endpoints will be rethrown for you to manually catch when performing internal requests.

```php
public function store()
{
    throw new Symfony\Component\HttpKernel\Exception\ConflictHttpException('We got a conflict!');
}
```

If we were to internally call the above endpoint we'd need to catch the exception thrown.

```php
try {
    app('Dingo\Api\Dispatcher')->with($payload)->post('users');
} catch (Symfony\Component\HttpKernel\Exception\ConflictHttpException $e) {
    // Do something here, like return with an error.
}
```

The package may also throw a `Dingo\Api\Exception\InternalHttpException` when an error response is returned. If you're using the response builder to return errors then you'll need to catch the thrown exception. The response will be available on the exception.

```php
public function show($id)
{
    return $this->response->errorNotFound('Could not find the user.');
}
```

When internally calling this endpoint we'll need to catch the exception.

```php
try {
    app('Dingo\Api\Dispatcher')->get('users/1');
} catch (Dingo\Api\Exception\InternalHttpException $e) {
    // We can get the response here to check the status code of the error or response body.
    $response = $e->getResponse();
}
```

[← Authentication](https://github.com/dingo/api/wiki/Authentication) | [OAuth 2.0 →](https://github.com/dingo/api/wiki/OAuth-2.0)
