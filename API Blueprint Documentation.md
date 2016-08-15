Documenting your API is as important as having a functional API. To help make the documenting process easier this package allows you to annotate your API controllers and then generate documentation using the Artisan command line utility.

### Artisan Command

To generate documentation you can use the `api:docs` command. The command has two required arguments, the name of the documentation and the version to generate.

For usage details see the [Commands](https://github.com/dingo/api/wiki/Commands#apidocs) section.

### Resources

Controllers can generally be represented as a resource. Resources may contain a number of actions which are represented by HTTP verbs.

To define a resource we can use the `@Resource` annotation.

```php
/**
 * @Resource("Users")
 */
class UserController extends Controller
{

}
```

The first parameter given to the resource is its identifier or the name of the resource. We can also give a base URI to the resource.

```php
/**
 * @Resource("Users", uri="/users")
 */
class UserController extends Controller
{

}
```

You can also provide a description of your resource prior to the annotation.

```php
/**
 * User resource representation.
 *
 * @Resource("Users", uri="/users")
 */
class UserController extends Controller
{

}
```

### Actions

An action is represented by a routable method on your controller.

You can describe your action with the short and long descriptions of a PHPDoc.

```php
/**
 * Show all users
 *
 * Get a JSON representation of all the registered users.
 */
public function index()
{

}
```

#### `@Get`, `@Post`, `@Put`, `@Patch`, `@Delete`

Each action is represented by an HTTP verb. You must provide a URI as the first parameter to the annotation.

```php
/**
 * Show all users
 *
 * Get a JSON representation of all the registered users.
 *
 * @Get("/")
 */
public function index()
{

}
```

#### `@Versions`

An action may be available across multiple API versions. When generating documentation this annnotation is used to determine what actions will be included once generated.

```php
/**
 * Show all users
 *
 * Get a JSON representation of all the registered users.
 *
 * @Get("/")
 * @Versions({"v1"})
 */
public function index()
{

}
```

#### `@Request`

An action should define a request that can be executed that will result in a successful or unsuccessful response.

A request should contain a body. Depending on the type of request the body will vary. For `POST` requests you can use a string, however you will also need to set the content type.

```php
/**
 * Register user
 *
 * Register a new user with a `username` and `password`.
 *
 * @Post("/")
 * @Versions({"v1"})
 * @Request("username=foo&password=bar", contentType="application/x-www-form-urlencoded")
 */
public function store()
{

}
```

If you're sending JSON you can use an annotation array and it will automatically be encoded to a JSON string. The content type will default to `application/json`.

```php
/**
 * Register user
 *
 * Register a new user with a `username` and `password`.
 *
 * @Post("/")
 * @Versions({"v1"})
 * @Request({"username": "foo", "password": "bar"})
 */
public function store()
{

}
```

You can also include additional headers.

```php
/**
 * Register user
 *
 * Register a new user with a `username` and `password`.
 *
 * @Post("/")
 * @Versions({"v1"})
 * @Request({"username": "foo", "password": "bar"}, headers={"X-Custom": "FooBar"})
 */
public function store()
{

}
```

If your action responds differently to multiple requests you must also identify the request.

```php
/**
 * Register user
 *
 * Register a new user with a `username` and `password`.
 *
 * @Post("/")
 * @Versions({"v1"})
 * @Request({"username": "foo", "password": "bar"}, identifier="A")
 */
public function store()
{

}
```

#### `@Response`

An `@Request` should always be followed by an `@Response` which defines the status code along with the content type, body, and headers.

Much like a request the response body can be a string (make sure to change the `contentType`) or an annotation array which will be encoded as JSON.

```php
/**
 * Register user
 *
 * Register a new user with a `username` and `password`.
 *
 * @Post("/")
 * @Versions({"v1"})
 * @Request({"username": "foo", "password": "bar"})
 * @Response(200, body={"id": 10, "username": "foo"})
 */
public function store()
{

}
```

Like a request you can also include headers.

#### `@Transaction`

A transaction lets you define multiple requests and multiple responses for requests. Responses must follow requests however you can define multiple responses for a single request.

```php
/**
 * Register user
 *
 * Register a new user with a `username` and `password`.
 *
 * @Post("/")
 * @Versions({"v1"})
 * @Transaction({
 *      @Request({"username": "foo", "password": "bar"}),
 *      @Response(200, body={"id": 10, "username": "foo"}),
 *      @Response(422, body={"error": {"username": {"Username is already taken."}}})
 * })
 */
public function store()
{

}
```

#### `@Parameters`

If your URI contains query string parameters you can define them either at the resource level or the action level. If a parameter is defined at the resource level you will need to either define it for each action or on the resource.

```php
/**
 * Show all users
 *
 * Get a JSON representation of all the registered users.
 *
 * @Get("/{?page,limit}")
 * @Versions({"v1"})
 * @Parameters({
 *      @Parameter("page", description="The page of results to view.", default=1),
 *      @Parameter("limit", description="The amount of results per page.", default=10)
 * })
 */
public function index()
{

}
```

You can also define the parameters `type` and whether or not it's `required`.

```php
/**
 * @Parameters({
 *      @Parameter("example", type="integer", required=true description="This is an example.", default=1)
 * })
 */
public function index()
{

}
```

[← Making Requests To Your API](https://github.com/dingo/api/wiki/Making-Requests-To-Your-API) | [Commands →](https://github.com/dingo/api/wiki/Commands)
