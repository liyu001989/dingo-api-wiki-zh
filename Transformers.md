Transformers allow you to easily and consistently transform objects into an array. By using a transformer you can type-cast integers and booleans, include pagination results, and nest relationships.

Transformers 允许你简单、统一的把对象转换为数组。通过使用一个 transformer 你可以类型转换整数、布尔值、分页结果、嵌套关系。

### Terminology 专业术语

The word "transformer" is used quite a bit in this chapter. It's worth noting what the following terms mean when used throughout this chapter.

在这一章 "transformer" 这个词用的比较多。需要注意下面的词语在使用的时候是什么意思，这贯穿了整个章节。

- The **transformation layer** is a library that prepares and handles transformers.
- A **transformer** is a class that will takes raw data and returns it as a presentable array ready for formatting. The way a transformer is handled is dependant upon the transformation layer.

- **转换层** 是一个类库，准备和操作 transformers。
- 一个 *transformer* 是一个类，它把原始的数据 。转换层决定了一个 transformer 的处理方式。

### Using Transformers 使用 Transformers

There are a couple ways to make use of transformers.

这有两种方法去使用 transformers。

#### Register A Transformer For A Class 为一个类注册一个 Transformer

When you register a transformer for a githubiven class you'll be able to return the class (assuming it can be turned into an array) directly from your routes and it'll
be run through the transformer automatically. This is great for simple APIs where you're using models as you can simply return the model from your route.

当你给一个类注册 transformer 的时候, 可以从你的路由直接返回这个类 （假设它能被转换为数组），它将会自动的通过 transformer 转换。这很有利于简单的 API，在你使用model的地方，你可以从路由简单的返回一个模型。

```php
app('Dingo\Api\Transformer\Factory')->register('User', 'UserTransformer');
```

#### Use The Response Builder 使用响应生成器

Refer to the [Response Builder](https://github.com/liyu001989/dingo-api-wiki-zh/blob/master/Responses.md#response-builder-响应生成器) chapter for help on how to do this.

返回 [响应生成器](https://github.com/liyu001989/dingo-api-wiki-zh/blob/master/Responses.md#response-builder-响应生成器) 小节，查看如何使用。

### Fractal

[Fractal](http://fractal.thephpleague.com) is the default transformation layer used by the package. It boasts a number of useful features to help you keep your data consistent.

[Fractal](http://fractal.thephpleague.com) 是这个包默认的转换层。

To use Fractal it's recommend you read through the documentation found on their website.

要使用 Fractal，建议你去他们的网站阅读文档。

#### Automatic Relationship Eager-loading 自动关系预加载

When using Fractal's includes feature to embed relationships you should ensure you name them the same as the relationships on your models. This package will automatically eager-load those relationships for you to save on query counts.

当使用 Fractal 的引入功能嵌入关系的时候，你应该确保你在你的模型中定义了相同的名字。这个包会自动为你预加载这些关系，以减少查询数量。

#### Advanced Configuration 更多的配置

Fractal is registered as the default transformation layer and with the default options. To configure the include key and separator used when defining relationships to embed you must manually instantiate the `Dingo\Api\Transformer\Adapter\Fractal` instance in a service provider or bootstrap file.

Fractal 被注册为默认的转换层，用默认的配置。当定义嵌入的关系的时候，如果要配置引入的关键字和使用的分隔符，你必须在 service provider 或启动文件中手动实例化 `Dingo\Api\Transformer\Adapter\Fractal`。

```php
$this->app['Dingo\Api\Transformer\Factory']->setAdapter(function ($app) {
     return new Dingo\Api\Transformer\Adapter\Fractal(new League\Fractal\Manager, 'include', ',');
});
```

If you're using Lumen you can do this in your bootstrap file.

如果你正在使用 Lumen 你可以在启动文件里面这么做。

```php
app('Dingo\Api\Transformer\Factory')->setAdapter(function ($app) {
    return new Dingo\Api\Transformer\Adapter\Fractal(new League\Fractal\Manager, 'include', ',');
});
```

#### Advanced Usage With Response Builder

Using Fractal in conjunction with the response builder is typically the best way to return your data from controllers.

从控制器返回数据的最佳方式是将响应生成器和 Fractal 结合。

The `item`, `collection`, and `paginator` methods on the response builder all receive a few extra parameters that can be used to further customize Fractal.

响应生成器中的 `item`, `collection` 和  `paginator` 方法，都会接受一些额外的参数，这些参数可以进一步的自定义 Fractal。

##### Resource Key 资源关键字

```php
return $this->item($user, new UserTransformer, ['key' => 'user']);
```

##### Utilizing The Callback 利用回调

The Fractal transformation layer allows you to register a callback that is fired after the creation of the resource. This callback receives either an instance of `League\Fractal\Resource\Item` or `League\Fractal\Resource\Collection`as its first parameter and an instance of `League\Fractal\Manager` as the second. You can then use this to interact with the resource at a much more complex level.

Fractal 转换成允许你注册一个回调，这个回调将在资源创建之后被调用。这个回调接受 `League\Fractal\Resource\Item` 或 `League\Fractal\Resource\Collection` 的实例作为第一个参数，一个 `League\Fractal\Manager` 实例作为第二个参数。然后你就可以用它在更复杂的一层与资源交互。

The most obvious use case is to set the cursor implementation when wanting to paginate data or to change the serializer on a per response basis.

最直接的例子是为分页数据设置游标或再每个响应的基础上修改 serializer。

```php
return $this->collection($users, new UserTransformer, [], function ($resource, $fractal) {
    $resource->setCursor($cursor);
});
```

If you're not using the parameters to pass a resource key you can omit the empty array and simply pass the callback as the third parameter.

如果你不使用参数传递资源关键字，你可以省略空数组，简单的传递回调作为第三个参数。

```php
return $this->collection($users, new UserTransformer, function ($resource, $fractal) {
    $fractal->setSerializer(new CustomSerializer);
});
```

### Custom Transformation Layer 自定义转换层

Should you need a more custom approach to how your data is transformed then you can easily implement your transformation layer with this package. You'll need to create a class that implements `Dingo\Api\Contract\Transformer\Adapter` and has the required `transform` method.

如果你需要一个更定制化的方式，那么你可以轻松的的实现你自己的转换层。你需要创建一个实现了 `Dingo\Api\Contract\Transformer\Adapter`  的类，并且该类有 `transform` 方法。

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

这个 `transform` 是唯一需要的方法，你可以自由的添加你喜欢的其他方法。`transform`  方法的目的是带着 `$response`， 然后在转换层随着 `$transformer` 关闭。你的转换层需要返回一个数组，最后被 `transform` 方法返回。当然，如果你的这层非常简单，你可以完全在这个类中做这些。（没懂）

The `$binding` parameter may be useful should your transformation layer include more advanced features such as the ability to add meta data or allow other developers to interact with your layer via the callback.

`$binding` 参数会很有用，当你的转换层包含了更多高级的功能，比如添加 meta 数据或者允许其他开发者利用回调与你的这一层交流。

The `$request` parameter is the current HTTP request being executed and is useful when your transformation layer requires query string values or other related data.

`$request` 参数是当前正在执行的 HTTP request，当你的转换层需要查询值或者其他相关数据的时候会很有用。

[← Errors And Error Responses](https://github.com/liyu001989/dingo-api-wiki-zh/blob/master/Errors-And-Error-Responses.md) | [Authentication →](https://github.com/liyu001989/dingo-api-wiki-zh/blob/master/Authentication.md)
