A functioning API is built upon receiving requests and returning a consumable response to the client. An API will generally return responses in an easy to consume format such as JSON. There's a number of different ways to return responses and it will largely depend on how complex your API is or will become.

The easiest way to return a consumable response is to simply return an array or object directly from your controller. Not every object can be correctly formatted though so you should ensure that it implements either the `ArrayObject` or the `Illuminate\Support\Contracts\ArrayableInterface` interface.

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

### Response Builder

The response builder provides a fluent interface to easily build a more customizable response. The response builder is generally used in conjunction with **transformers**.

To utilize the response builder your controllers should use the `Dingo\Api\Routing\Helpers` trait. To save importing and using the trait on all your controllers you can simply create a base controller that all your API controllers will extend.

```php
use Dingo\Api\Routing\Helpers;
use Illuminate\Routing\Controller;

class BaseController extends Controller
{
	use Helpers;
}
```

Now your controllers can simply extend this base controller. The response builder is available via the `$response` property on your controller. Note that all the methods below can also be used as `withX` should you prefer that syntax.

> Some of the documentation below makes use of [Transformers](https://github.com/dingo/api/wiki/Transformers), be sure to read that chapter for more details.

#### Responding With An Array

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

#### Responding With A Single Item

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

#### Responding With A Collection Of Items

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

#### Responding With Paginated Items

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

#### Responding With No Content

```php
return $this->response->noContent();
```

#### Responding With Created Response

```php
return $this->response->created();
```

You can also optionally provide a location for the created resource as the first paramter.

```php
return $this->response->created($location);
```

#### Responding With An Error

There's a number of different built in errors you can use to quickly generate an error response.

```php
// A generic error with custom message and status code.
return $this->response->error('This is an error.', 404);

// A not found error with an optional message as the first parameter.
return $this->response->errorNotFound();

// A bad request error with an optional message as the first parameter.
return $this->response->errorBadRequest();

// A forbidden error with an optional message as the first parameter.
return $this->response->errorForbidden();

// An internal error with an optional message as the first parameter.
return $this->response->errorInternal();

// An unauthorized error with an optional message as the first parameter.
return $this->response->errorUnauthorized();
```

#### Adding Additional Headers

Once you've used one of the above methods you can further customize the response by adding additional headers.

```php
return $this->response->item($user, new UserTransformer)->withHeader('X-Foo', 'Bar');
```

#### Adding Meta Data

Some transformation layers may utilize meta data. This is useful when you need to provide extra data that's associated with the resource.

```php
return $this->response->item($user, new UserTransformer)->addMeta('foo', 'bar');
```

You can also set an array of meta data instead of chaining multiple method calls.

```php
return $this->response->item($user, new UserTransformer)->setMeta($meta);
```

#### Setting Response Status Code

```php
return $this->response->item($user, new UserTransformer)->setStatusCode(200);
```

### Custom Response Formats

In the **configuration** chapter we briefly touched on response formats. By default the package will automatically use the JSON format and set an appropriate `Content-Type` header. Aside from a JSON formatter there is also a JSONP formatter. This formatter will wrap the responses in a callback. To register this format you can simply swap out the default JSON formatter in the configuration file or in your bootstrap file.

```php
'formats' => [
    'json' => 'Dingo\Api\Http\Response\Format\Jsonp'
]
```

```php
Dingo\Api\Http\Response::addFormatter('json', new Dingo\Api\Http\Response\Format\Jsonp);
```

By default the callback parameter expected in the query string is `callback`, this can be changed by passing in the first parameter to the class constructor. If the query string does not contain a parameter with the name of your callback parameter it will default to a JSON response.

You can also register and use your own formatters should you need to. Your formatter should extend `Dingo\Api\Http\Response\Format\Format`. There following methods should be defined: `formatEloquentModel`, `formatEloquentCollection`, `formatArray`, and `getContentType`. Refer to the abstract class for more details on what each method should do or take a look at the pre-defined format classes.

### Morphing And Morphed Events

Before this package sends a response it will first `morph` the response. This process involves running any transformers as well as sending the response through the configured formatter. If you need more control of how a response is morphed you can use the `ResponseWasMorphed` and `ResponseIsMorphing` events.

Create a listener for either event in your `app/Listeners` directory.

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

```php
protected $listen = [
    'Dingo\Api\Event\ResponseWasMorphed' => [
    	'App\Listeners\AddPaginationLinksToResponse'
    ]
];
```

Now all responses that contain pagination links will also add these links to the `Link` header.

> Note that this code isn't production ready and is merely a demonstration of how you can utilize the events.

[← Creating API Endpoints](https://github.com/dingo/api/wiki/Creating-API-Endpoints) | [Errors And Error Responses →](https://github.com/dingo/api/wiki/Errors-And-Error-Responses)
