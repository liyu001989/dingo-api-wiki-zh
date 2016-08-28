A functioning API is built upon receiving requests and returning a consumable response to the client. An API will generally return responses in an easy to consume format such as JSON. There's a number of different ways to return responses and it will largely depend on how complex your API is or will become.

一个功能健全的 API 是建立在接受消息和返回可用的响应给可客户端之上。一个 API 一般会返回一个易于格式化的响应，例如 JSON。这里有很多不同的方法返回响应，这取决于你的 API 有多么的复杂或者要成为什么样。

The easiest way to return a consumable response is to simply return an array or object directly from your controller. Not every object can be correctly formatted though so you should ensure that it implements either the `ArrayObject` or the `Illuminate\Support\Contracts\ArrayableInterface` interface.

最简单的方法返回可用的响应是在 controller 中直接返回数组或者对象。不是每一个对象都可以直接格式化，所以你需要确保它实现了 `ArrayObject` 或者 `Illuminate\Support\Contracts\ArrayableInterface` 接口。

```php
class UserController
{
	public function index()
	{
		return User::all();
	}
}
```

In this case our `User` class extends from `Illuminate\Database\Eloquent\Model` which means it can be formatted as an array, so we can simply return the collection of users we get from calling `User::all()`. Similarly you can use it return a single user.

再这个例子里，我们的 `User` 类继承了 `Illuminate\Database\Eloquent\Model` 意味着他能被格式化成数组，所以我们可以直接返回 users 的集合，通过调用 `User::all()`。同样的你可以返回一个单一的用户。

```php
class UserController
{
	public function show($id)
	{
		return User::findOrFail($id);
	}
}
```

The package will automatically format the response as JSON and set the `Content-Type` header to `application/json`.

这个包将会自动的格式化响应为 JSON，并设置 `Content-Type` 头为 `application/json`。

### Response Builder 建造响应

The response builder provides a fluent interface to easily build a more customizable response. The response builder is generally used in conjunction with **transformers**.

响应的 builder 提供了一个流畅的接口去简单的建立一个更定制化的响应。响应的 builder 通常是与 **transformer**相结合。

To utilize the response builder your controllers should use the `Dingo\Api\Routing\Helpers` trait. To save importing and using the trait on all your controllers you can simply create a base controller that all your API controllers will extend.

要利用响应 builder 你的控制器需要使用 `Dingo\Api\Routing\Helpers` trait。为了保持引入和使用这个 trait 在你的控制器里，你可以简单的创建一个基础控制器，然后你的所有的 API 控制器都继承它。

```php
use Dingo\Api\Routing\Helpers;
use Illuminate\Routing\Controller;

class BaseController extends Controller
{
	use Helpers;
}
```

Now your controllers can simply extend this base controller. The response builder is available via the `$response` property on your controller. Note that all the methods below can also be used as `withX` should you prefer that syntax.

现在你的控制器可以直接继承基础控制器。响应 builder 可以在控制器里通过 `$response` 属性获取。提示，下面所有的方法也可以用做 `withX` 使用，你应该会更喜欢这种语法。

> Some of the documentation below makes use of [Transformers](https://github.com/liyu001989/dingo-api-wiki-zh/blob/master/Transformers.md), be sure to read that chapter for more details.

> 下面的一些文档利用了 [Transformers](https://github.com/liyu001989/dingo-api-wiki-zh/blob/master/Transformers.md)，请务必阅读这一章了解更多详情。

#### Responding With An Array 响应一个数组

```php
class UserController extends BaseController
{
	public function show($id)
	{
		$user = User::findOrFail($id);

		return $this->response->array($user->toArray());
	}
}
```

#### Responding With A Single Item 响应一个元素

```php
class UserController extends BaseController
{
	public function show($id)
	{
		$user = User::findOrFail($id);

		return $this->response->item($user, new UserTransformer);
	}
}
```

#### Responding With A Collection Of Items 响应一个元素集合

```php
class UserController extends BaseController
{
	public function index()
	{
		$users = User::all();

		return $this->response->collection($users, new UserTransformer);
	}
}
```

#### Responding With Paginated Items 响应分页

```php
class UserController extends BaseController
{
	public function index()
	{
		$users = User::paginate(25);

		return $this->response->paginator($users, new UserTransformer);
	}
}
```

#### Responding With No Content 无内容响应

```php
return $this->response->noContent();
```

#### Responding With Created Response 创建资源的响应

```php
return $this->response->created();
```

You can also optionally provide a location for the created resource as the first paramter.

你可以现则在第一个参数的位置，提供创建的元素的位置。

```php
return $this->response->created($location);
```

#### Responding With An Error 错误响应

There's a number of different built in errors you can use to quickly generate an error response.

这有很多不同的方式创建错误响应，你可以快速的生成一个错误响应。

```php
// A generic error with custom message and status code.
// 一个自定义消息和状态码的普通错误。
return $this->response->error('This is an error.', 404);

// A not found error with an optional message as the first parameter.
// 一个没有找到资源的错误，第一个参数可以传递自定义消息。
return $this->response->errorNotFound();

// A bad request error with an optional message as the first parameter.
// 一个 bad request 错误，第一个参数可以传递自定义消息。
return $this->response->errorBadRequest();

// A forbidden error with an optional message as the first parameter.
// 一个服务器拒绝错误，第一个参数可以传递自定义消息。
return $this->response->errorForbidden();

// An internal error with an optional message as the first parameter.
// 一个网络错误，第一个参数可以传递自定义消息。
return $this->response->errorInternal();

// An unauthorized error with an optional message as the first parameter.
// 一个未认证错误，第一个参数可以传递自定义消息。
return $this->response->errorUnauthorized();
```

#### Adding Additional Headers 添加额外的头信息

Once you've used one of the above methods you can further customize the response by adding additional headers.

一旦你已经使用了上面的方法，你可以自己添加额外的头信息

```php
return $this->response->item($user, new UserTransformer)->withHeader('X-Foo', 'Bar');
```

#### Adding Meta Data 添加 Meta 信息

Some transformation layers may utilize meta data. This is useful when you need to provide extra data that's associated with the resource.

一些数据转换层也许利用了 meta 数据。当你需要提供链接资源的额外数据这将会很有用。

```php
return $this->response->item($user, new UserTransformer)->addMeta('foo', 'bar');
```

You can also set an array of meta data instead of chaining multiple method calls.

你也可以设置一个 meta 数据的数组，代替多次调用多次 addMeta 方法。

```php
return $this->response->item($user, new UserTransformer)->setMeta($meta);
```

#### Setting Response Status Code 设置响应状态码

```php
return $this->response->item($user, new UserTransformer)->setStatusCode(200);
```

### Custom Response Formats 自定义响应格式

In the **configuration** chapter we briefly touched on response formats. By default the package will automatically use the JSON format and set an appropriate `Content-Type` header. Aside from a JSON formatter there is also a JSONP formatter. This formatter will wrap the responses in a callback. To register this format you can simply swap out the default JSON formatter in the configuration file or in your bootstrap file.

在 **配置** 这章，我们大概的接触了响应格式化。默认的，这个包会自动使用 JSON 格式并设置相关的 `Content-Type` 头。除了 JSON 格式化，还有一个 JSONP 格式化。这个 formatter 将会用一个回调包裹响应。注册这个格式化你可以简单的替换 JSON formatter 在配置文件或者你的启动文件中。

```php
'formats' => [
    'json' => 'Dingo\Api\Http\Response\Format\Jsonp'
]
```

```php
Dingo\Api\Http\Response::addFormatter('json', new Dingo\Api\Http\Response\Format\Jsonp);
```

By default the callback parameter expected in the query string is `callback`, this can be changed by passing in the first parameter to the class constructor. If the query string does not contain a parameter with the name of your callback parameter it will default to a JSON response.

默认情况下，预计的 query string 中的回调参数是 `callback`，可以传递第一个参数到 class 的构造函数中去替换。如果 query string 中没有提供回调参数的名字，它将默认的返回 JSON 响应。（我没看懂，求修正）

You can also register and use your own formatters should you need to. Your formatter should extend `Dingo\Api\Http\Response\Format\Format`. There following methods should be defined: `formatEloquentModel`, `formatEloquentCollection`, `formatArray`, and `getContentType`. Refer to the abstract class for more details on what each method should do or take a look at the pre-defined format classes.

如果你需要，你可以注册并使用你自己的 formatters。你的 formatter 需要继承 `Dingo\Api\Http\Response\Format\Format`。下面的方法需要被定义：`formatEloquentModel`, `formatEloquentCollection`, `formatArray` 和 `getContentType`。你可以在预定义格式化类中得到更多的资料，包括提到的抽象类中每个方法需要做什么。

### Morphing And Morphed Events

Before this package sends a response it will first `morph` the response. This process involves running any transformers as well as sending the response through the configured formatter. If you need more control of how a response is morphed you can use the `ResponseWasMorphed` and `ResponseIsMorphing` events.

在这个包返回响应之前，会首先会 `改变` 响应。这个过程涉及到运行所有 transformers 以及发送满足配置的格式化响应。如果你需要对响应的变化有更多的控制，你可以使用 `ResponseWasMorphed` 和 `ResponseIsMorphing` 这两个事件.

Create a listener for either event in your `app/Listeners` directory.

在 `app/Listeners` 目录中为这两个事件创建监听者。

```php
use Dingo\Api\Event\ResponseWasMorphed;

class AddPaginationLinksToResponse
{
	public function handle(ResponseWasMorphed $event)
	{
		if (isset($event->content['meta']['pagination'])) {
			$links = $event->content['meta']['pagination']['links'];

			$event->response->headers->set(
				'link',
				sprintf('<%s>; rel="next", <%s>; rel="prev"', $links['links']['next'], $links['links']['previous'])
			);
		}
	}
}
```

You can then listen for the events by adding the event and your listener in your `EventServiceProvider`.

你可以通过添加事件和监听者到你的 `EventServiceProvider` 中来监听这个事件。

```php
protected $listen = [
    'Dingo\Api\Event\ResponseWasMorphed' => [
    	'App\Listeners\AddPaginationLinksToResponse'
    ]
];
```

Now all responses that contain pagination links will also add these links to the `Link` header.

现在，所有的响应都会包含分页链接在，这些链接会放在 `Link` 头中。

> Note that this code isn't production ready and is merely a demonstration of how you can utilize the events.

> 提示，这并不是生产环境的代码，它仅仅只是怎么利用事件的例子。

[← Creating API Endpoints](https://github.com/liyu001989/dingo-api-wiki-zh/blob/master/Creating-API-Endpoints.md) | [Errors And Error Responses →](https://github.com/liyu001989/dingo-api-wiki-zh/blob/master/Errors-And-Error-Responses.md)
