Making a request to your API is quite simple. The best way to do this is using a tool such as [Postman](http://www.getpostman.com/).

请求你的 API 非常的简单。最好的方法是使用工具，比如 [Postman](http://www.getpostman.com/)。

Because we aren't versioning the API in the URI we need to define an `Accept` header to request a specific version. The header is formatted like so.

因为我们不在 URI 中指定具体的版本，所以我们需要定义 `Accept` 头去请求具体的版本。头信息格式如下。

```
Accept: application/vnd.YOUR_SUBTYPE.v1+json
```

In the above example you would replace `YOUR_SUBTYPE` with the subtype name you defined in your configuration. Again, this is usually something unique to your application, such as its name or identifier, and is usually all lowercase.

再上面的例子中，你需要用你配置中定义的子类型名字替换 `YOUR_SUBTYPE`。而且，这是你的应用特有的东西，比如他的名字或id，通常都是小写。

> Remember if you use a different standards tree, such as `x`, then you'd replace `vnd` with `x`.

> 记住如果你使用不同的标准树，比如  `x` 那么你需要替换 `vnd` 为 `x`.

Following the subtype name we have the version we want. In the above example we're requesting `v1` of our API. This is then followed by a plus sign and the desired format. If the format is invalid the package will attempt to use the default format you defined in your configuration.

随着子类型名称，我们有了我们想要的版本。再上面的例子中，我们正在请求 `v1` 版本的 API。随后是一个加号和规定的格式。如果格式错误，dingo/api 会尝试使用配置文件中的默认格式。

If you don't want to use Postman you can use a command line tool such as cURL.

如果你不想使用 Postman，你可以使用命令行工具，比如 cURL。

```
$ curl -v -H "Accept: application/vnd.YOUR_SUBTYPE.v1+json" http://example.app/users
```

If you have strict mode enabled and you pass an invalid `Accept` header an unhandled `Symfony\Component\HttpKernel\Exception\BadRequestHttpException` will be thrown.

如果你使用了严格模式，而且你传递了一个错误的 `Accept` 头，那么一个未处理的 `Symfony\Component\HttpKernel\Exception\BadRequestHttpException` 异常将被抛出。

[← OAuth 2.0](https://github.com/liyu001989/dingo-api-wiki-zh/blob/master/OAuth-2.0.md) | [API Blueprint Documentation →](https://github.com/liyu001989/dingo-api-wiki-zh/blob/master/API-Blueprint-Documentation.md)
