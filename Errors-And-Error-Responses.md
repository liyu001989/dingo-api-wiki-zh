Dealing with errors when building an API can be a pain. Instead of manually building error responses you can simply throw an exception that extends from `Symfony\Component\HttpKernel\Exception\HttpException` and the API will automatically handle the response for you.

创建一个 API 的时候处理错误是很痛苦的。为了避免手动的创建错误响应，你可以简单的抛出一个继承了 `Symfony\Component\HttpKernel\Exception\HttpException` 的异常，API 会自动的为你处理响应。

Here is a list of built-in Symfony exceptions.

这里是 Symfony 内置的异常列表。

| Exception                                                                | Status Code |
|--------------------------------------------------------------------------|------------:|
| `Symfony\Component\HttpKernel\Exception\AccessDeniedHttpException`         |         403 |
| `Symfony\Component\HttpKernel\Exception\BadRequestHttpException`           |         400 |
| `Symfony\Component\HttpKernel\Exception\ConflictHttpException`             |         409 |
| `Symfony\Component\HttpKernel\Exception\GoneHttpException`                 |         410 |
| `Symfony\Component\HttpKernel\Exception\HttpException`                     |         500 |
| `Symfony\Component\HttpKernel\Exception\LengthRequiredHttpException`       |         411 |
| `Symfony\Component\HttpKernel\Exception\MethodNotAllowedHttpException`     |         405 |
| `Symfony\Component\HttpKernel\Exception\NotAcceptableHttpException`        |         406 |
| `Symfony\Component\HttpKernel\Exception\NotFoundHttpException`             |         404 |
| `Symfony\Component\HttpKernel\Exception\PreconditionFailedHttpException`   |         412 |
| `Symfony\Component\HttpKernel\Exception\PreconditionRequiredHttpException` |         428 |
| `Symfony\Component\HttpKernel\Exception\ServiceUnavailableHttpException`   |         503 |
| `Symfony\Component\HttpKernel\Exception\TooManyRequestsHttpException`      |         429 |
| `Symfony\Component\HttpKernel\Exception\UnauthorizedHttpException`         |         401 |
| `Symfony\Component\HttpKernel\Exception\UnsupportedMediaTypeHttpException` |         415 |

As an example you might throw a `ConflictHttpException` when you attempt to update a record that has been updated by another user prior to this update request.

举个例子，当你修改一个记录后，其他的用户已经在你之前修改了它，你可能会抛出一个 `ConflictHttpException` 异常。

```php
$api->version('v1', function ($api) {
    $api->put('user/{id}', function ($id) {
        $user = User::find($id);

        if ($user->updated_at > app('request')->get('last_updated')) {
            throw new Symfony\Component\HttpKernel\Exception\ConflictHttpException('User was updated prior to your request.');
        }

        // No error, we can continue to update the user as per usual.
    });
});
```

The package automatically catches the thrown exception and will convert it into its JSON representation. The responses HTTP status code is also changed to match that of the exception. A `ConflictHttpException` would result in an HTTP 409 status code and the following JSON representation assuming you haven't changed the default error format.

这个包自动的接住异常，然后转换为 JSON。响应的 HTTP 状态码也会根据异常而改变。如果你没有改变默认的错误格式，一个 `ConflictHttpException` 异常返回的结果为，HTTP 409 状态码和响应的 JSON 表述。

```json
{
	"message": "User was updated prior to your request.",
	"status_code": 409
}
```

### Resource Exceptions 资源异常

The following is a list of generic resource exceptions. Each of these returns an HTTP 422 status code.

下面是通用的资源异常的列表，它们都会放回 HTTP 422 状态码。

```php
Dingo\Api\Exception\DeleteResourceFailedException
Dingo\Api\Exception\ResourceException
Dingo\Api\Exception\StoreResourceFailedException
Dingo\Api\Exception\UpdateResourceFailedException
```

These exceptions are special in that they allow you to pass along any validation errors that occurred when trying to create, update, or delete resources.

这些异常是特殊的，因为它们允许你传递任何验证错误，当你尝试去创建、更新或者删除资源的时候。

As an example you might throw a `StoreResourceFailedException` when you encounter errors when trying to validate the creation of a new user.

```php
$api->version('v1', function ($api) {
    $api->post('users', function () {
        $rules = [
            'username' => ['required', 'alpha'],
            'password' => ['required', 'min:7']
        ];

        $payload = app('request')->only('username', 'password');

        $validator = app('validator')->make($payload, $rules);

        if ($validator->fails()) {
            throw new Dingo\Api\Exception\StoreResourceFailedException('Could not create new user.', $validator->errors());
        }

        // Create user as per usual.
    });
});
```

The package automatically catches the thrown exception and will convert it into its JSON representation. The responses HTTP status code is also changed to match that of the exception. Resources exceptions result in an HTTP 422 status code and the following JSON representation.

这个包自动的抓取这些异常，转换为 JSON 的表述。响应的 HTTP 状态码也会根据异常而改变。资源异常返回的结果为 HTTP 422 状态码和响应的 JSON 表述。

```json
{
    "message": "Could not create new user.",
    "status_code": 422,
    "errors": {
        "username": [
            "The username field is required."
        ],
        "password": [
            "The password field is required."
        ]
    }
}
```

### Custom HTTP Exceptions 自定义 HTTP 异常

You can create your own custom HTTP exceptions so long as they extend from `Symfony\Component\HttpKernel\Exception\HttpException` or implement `Symfony\Component\HttpKernel\Exception\HttpExceptionInterface`.

你可以创建你自己的 HTTP 异常，只要它们继承了 `Symfony\Component\HttpKernel\Exception\HttpException` 或者实现了 `Symfony\Component\HttpKernel\Exception\HttpExceptionInterface` 接口。

### Custom Exception Responses 自定义异常响应

If you need to customize the response that exceptions return you can register a custom error handler.

如果你需要自定义异常的响应，你可以注册一个自定义错误 handler。

```php
app('Dingo\Api\Exception\Handler')->register(function (Symfony\Component\HttpKernel\Exception\UnauthorizedHttpException $exception) {
    return Response::make(['error' => 'Hey, what do you think you are doing!?'], 401);
});
```

Now if authentication fails we'll be presented with the following JSON representation.

现在如果验证错误，我们会用下面的 JSON 表述呈现。

```json
{
    "error": "Hey, what do you think you are doing!?"
}
```

### Form Requests 表单请求

If you're using form requests then you will need to either extend the base API form request class or implement your own. The base API form request class will
check to see if the incoming request is for the API, and, if it is, it will throw a `Dingo\Api\Exception\ValidationHttpException` if validation fails.

如果你正在使用表单请求，那么你不但需要继承基本的 API 表单请求类，还需要实现你自己的。基本的 API 表单请求类会检查当前的请求是否是为了API，如果是，当验证失败时将抛出 `Dingo\Api\Exception\ValidationHttpException` 异常。

This exception will then be rendered correctly by the package and the error response returned.

这个异常将会被正确的渲染，然后返回错误响应。

If you'd like to implement your own form request you *must* overload the `failedValidation` and `failedAuthorization` methods. These methods *must* throw one of the above mentioned exceptions and not the response HTTP exceptions that Laravel throws.

如果你喜欢实现你自己的表单请求，你 *必须* 重写 `failedValidation` 和 `failedAuthorization` 方法。这些方法 *必须* 抛出上面提到的异常中的一个，而且不是 Laravel 抛出的 HTTP 异常。

[← Responses](https://github.com/liyu001989/dingo-api-wiki-zh/blob/master/Responses.md) | [Transformers →](https://github.com/liyu001989/dingo-api-wiki-zh/blob/master/Transformers.md)
