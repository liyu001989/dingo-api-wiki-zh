Making a request to your API is quite simple. The best way to do this is using a tool such as [Postman](http://www.getpostman.com/).

Because we aren't versioning the API in the URI we need to define an `Accept` header to request a specific version. The header is formatted like so.

```
Accept: application/vnd.YOUR_SUBTYPE.v1+json
```

In the above example you would replace `YOUR_SUBTYPE` with the subtype name you defined in your configuration. Again, this is usually something unique to your application, such as its name or identifier, and is usually all lowercase.

> Remember if you use a different standards tree, such as `x`, then you'd replace `vnd` with `x`.

Following the subtype name we have the version we want. In the above example we're requesting `v1` of our API. This is then followed by a plus sign and the desired format. If the format is invalid the package will attempt to use the default format you defined in your configuration.

If you don't want to use Postman you can use a command line tool such as cURL.

```
$ curl -v -H "Accept: application/vnd.YOUR_SUBTYPE.v1+json" http://example.app/users
```

If you have strict mode enabled and you pass an invalid `Accept` header an unhandled `Symfony\Component\HttpKernel\Exception\BadRequestHttpException` will be thrown.

[← OAuth 2.0](https://github.com/dingo/api/wiki/OAuth-2.0) | [API Blueprint Documentation →](https://github.com/dingo/api/wiki/API-Blueprint-Documentation)
