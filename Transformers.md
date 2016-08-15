Transformers allow you to easily and consistently transform objects into an array. By using a transformer you can type-cast integers and booleans, include pagination results, and nest relationships.

### Terminology

The word "transformer" is used quite a bit in this chapter. It's worth noting what the following terms mean when used throughout this chapter.

- The **transformation layer** is a library that prepares and handles transformers.
- A **transformer** is a class that will takes raw data and returns it as a presentable array ready for formatting. The way a transformer is handled is dependant upon the transformation layer.

### Using Transformers

There are a couple ways to make use of transformers.

#### Register A Transformer For A Class

When you register a transformer for a given class you'll be able to return the class (assuming it can be turned into an array) directly from your routes and it'll
be run through the transformer automatically. This is great for simple APIs where you're using models as you can simply return the model from your route.

```php
app('Dingo\Api\Transformer\Factory')->register('User', 'UserTransformer');
```

#### Use The Response Builder

Refer to the [Response Builder](https://github.com/dingo/api/wiki/Responses#response-builder) chapter for help on how to do this.

### Fractal

[Fractal](http://fractal.thephpleague.com) is the default transformation layer used by the package. It boasts a number of useful features to help you keep your data consistent.

To use Fractal it's recommend you read through the documentation found on their website.

#### Automatic Relationship Eager-loading

When using Fractal's includes feature to embed relationships you should ensure you name them the same as the relationships on your models. This package will automatically eager-load those relationships for you to save on query counts.

#### Advanced Configuration

Fractal is registered as the default transformation layer and with the default options. To configure the include key and separator used when defining relationships to embed you must manually instantiate the `Dingo\Api\Transformer\Adapter\Fractal` instance in a service provider or bootstrap file.

```php
$this->app['Dingo\Api\Transformer\Factory']->setAdapter(function ($app) {
     return new Dingo\Api\Transformer\Adapter\Fractal(new League\Fractal\Manager, 'include', ',');
});
```

If you're using Lumen you can do this in your bootstrap file.

```php
app('Dingo\Api\Transformer\Factory')->setAdapter(function ($app) {
    return new Dingo\Api\Transformer\Adapter\Fractal(new League\Fractal\Manager, 'include', ',');
});
```

#### Advanced Usage With Response Builder

Using Fractal in conjunction with the response builder is typically the best way to return your data from controllers.

The `item`, `collection`, and `paginator` methods on the response builder all receive a few extra parameters that can be used to further customize Fractal.

##### Resource Key

```php
return $this->item($user, new UserTransformer, ['key' => 'user']);
```

##### Utilizing The Callback

The Fractal transformation layer allows you to register a callback that is fired after the creation of the resource. This callback receives either an instance of `League\Fractal\Resource\Item` or `League\Fractal\Resource\Collection`as its first parameter and an instance of `League\Fractal\Manager` as the second. You can then use this to interact with the resource at a much more complex level.

The most obvious use case is to set the cursor implementation when wanting to paginate data or to change the serializer on a per response basis.

```php
return $this->collection($users, new UserTransformer, [], function ($resource, $fractal) {
    $resource->setCursor($cursor);
});
```

If you're not using the parameters to pass a resource key you can omit the empty array and simply pass the callback as the third parameter.

```php
return $this->collection($users, new UserTransformer, function ($resource, $fractal) {
    $fractal->setSerializer(new CustomSerializer);
});
```

### Custom Transformation Layer

Should you need a more custom approach to how your data is transformed then you can easily implement your transformation layer with this package. You'll need to create a class that implements `Dingo\Api\Contract\Transformer\Adapter` and has the required `transform` method.

```php
use Dingo\Api\Http\Request;
use Dingo\Api\Transformer\Binding;
use Dingo\Api\Contract\Transformer\Adapter;

class MyCustomTransformer implements Adapter
{
    public function transform($response, $transformer, Binding $binding, Request $request)
    {
        // Make a call to your transformation layer to transformer the given response.
    }
}
```

This `transform` method is the only required method, you're free to add as many other methods as you like. The purpose of the `transform` method is to take the `$response`, and hand it off to your transformation layer along with the `$transformer`. Your transformation layer should then return an array which is then returned by the `transform` method. Of course, if your layer is very simple you can do it all inside of this class.

The `$binding` parameter may be useful should your transformation layer include more advanced features such as the ability to add meta data or allow other developers to interact with your layer via the callback.

The `$request` parameter is the current HTTP request being executed and is useful when your transformation layer requires query string values or other related data.

[← Errors And Error Responses](https://github.com/dingo/api/wiki/Errors-And-Error-Responses) | [Authentication →](https://github.com/dingo/api/wiki/Authentication)
