Documenting your API is as important as having a functional API. To help make the documenting process easier this package allows you to annotate your API controllers and then generate documentation using the Artisan command line utility.

为你的 API 添加文档跟完成一个 API 同样重要。为了使文档写起来更容易，这个包允许你在 controllers 中注释你的 API 然后通过 Artisan 命令生成文档。

### Artisan Command   Artisan 命令行

To generate documentation you can use the `api:docs` command. The command has two required arguments, the name of the documentation and the version to generate.

生成文档你可以使用 `api:docs` 命令。这个命令有两个必要的参数，文档的名字和生成的版本。

For usage details see the [Commands](https://github.com/liyu001989/dingo-api-wiki-zh/blob/master/Commands.md#apidocs) section.

更多使用详情见 [Commands](https://github.com/liyu001989/dingo-api-wiki-zh/blob/master/Commands.md#apidocs) 章。

### Resources 资源

Controllers can generally be represented as a resource. Resources may contain a number of actions which are represented by HTTP verbs.

控制机一般代表一个资源。资源可以包含若干代表 HTTP 动词的方法。

To define a resource we can use the `@Resource` annotation.

定义一个资源，我们可以使用 `@Resource` 注释。

```php
/**
 * @Resource("Users")
 */
class UserController extends Controller
{

}
```

The first parameter given to the resource is its identifier or the name of the resource. We can also give a base URI to the resource.

资源的第一个参数是它的标识符或者是资源的名字。我们也可以给一个资源基本的 URI。

```php
/**
 * @Resource("Users", uri="/users")
 */
class UserController extends Controller
{

}
```

You can also provide a description of your resource prior to the annotation.

我们也可以在注释的前面提供一段资源的注释。

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

一个方法代表控制器中的一个可被路由的方法。

You can describe your action with the short and long descriptions of a PHPDoc.

你可以通过短的或长的 PHPDoc 描述你的方法。

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

每个方法代表一个 HTTP 动词。你必须提供一个 URI 作为注释的第一个参数。

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

一个方法可以被访问到，通过不同版本的 API。当生成文档的时候，这个注释用来确定包含哪些方法。

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

一个方法需要定义一个请求，将导致一个成功或不成功的响应。

A request should contain a body. Depending on the type of request the body will vary. For `POST` requests you can use a string, however you will also need to set the content type.

一个请求需要包含一个body。根据请求的类型，body 会不同。对于 `POST` 请求你可以使用一个字符串，但是你依然需要设置 content type。

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

如果你发送 JSON 数据，你可以使用数组进行注释，它将自动的编码为 JSON 字符串。content type 将默认为 `application/json`。

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

你依然可以引入额外的头。

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

如果你的方法对不同的请求，会有不同的响应，那么你必须标识请求。

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

一个 `@Request` 也需要定义一个 `@Response`，它定义了状态码，以及content type, body 和 headers。

Much like a request the response body can be a string (make sure to change the `contentType`) or an annotation array which will be encoded as JSON.

像请求一样，响应的 body 可以是一个字符串(确保要更改 `contentType`) 或者一个JSON 数组的注释。

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

像请求一样，你依然可以引入头信息。

#### `@Transaction`

A transaction lets you define multiple requests and multiple responses for requests. Responses must follow requests however you can define multiple responses for a single request.

一个 transaction 允许你定义多个请求和多个响应。响应必须跟随请求，但你可以为一个请求定义多个响应。

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

如果你的 URI 包含查询参数，你可以在资源级别或者方法级别定义他们。如果一个参数定义在资源你级别，你需要在每个方法或资源定义它。

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

你也可以定义 `type` 参数，无论它是否 `必要`

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

[← Making Requests To Your API](https://github.com/liyu001989/dingo-api-wiki-zh/blob/master/Making-Requests-To-Your-API.md) | [Commands →](https://github.com/liyu001989/dingo-api-wiki-zh/blob/master/Commands.md)
