Much of the package is extendable in one way or another. Out of the box the core is configured for a general purpose use case and will usually satisfy the majority of users. If, however, you're finding yourself in need of implementing a different way of handling exceptions or using a separate request class, then you've come to the right place.

### Custom Request

If you'd like to implement your own custom request class instead of utilizing the default `Dingo\Api\Http\Request` class then you'll want to setup a new alias. Your request class should either extend from `Dingo\Api\Http\Request` (recommended) or implement the `Dingo\Api\Contract\Http\Request` interface. If you implement the interface then you'll be required to define the methods set out by the interface.

As an example, you may want to add some custom methods to the request.

```php
namespace MyApp;

use Dingo\Api\Http\Request as DingoRequest;

class Request extends DingoRequest
{
    public function hasSomething()
    {
        return $this->checkSomethingElse();
    }
}
```

To replace the request class simply set an alias on the application container in your service provider.

```php
$this->app->alias('MyApp\Request', 'Dingo\Api\Contract\Http\Request');
```

When the request instance is created it will now be an instance of your custom request class.

### Custom Request Validators

To determine if a request is targetting your application or the API this package performs some validation very early in the applications life cycle. This validation comes in the form of several validators:

```
Dingo\Api\Http\Validation\Accept
Dingo\Api\Http\Validation\Domain
Dingo\Api\Http\Validation\Prefix
```

The `Domain` and `Prefix` validators are used to determine if the request matches the domain or prefix defined in your configuration file. The `Accept` validator is always run after the two default validators and will parse the `Accept` header and, if configured, throw an exception if it is not defined correctly.

If you'd like to define your own validators you can do so by implementing the `Dingo\Api\Contract\Http\Validator\Validator` interface. The only requirement for this class is the `validate` method and should return either `true` or `false` depending on whether or not validation succeeded.

As an example let's say you'd like to check for the existence of some headers.

```
<?php

namespace MyApp\Http\Validation;

use Illuminate\Http\Request;
use Dingo\Api\Contract\Http\Validation\Validator;

class Headers implements Validator
{
    protected $headers = [];

    public function __construct(array $headers)
    {
        $this->headers = $headers;
    }

    public function validate(Request $request)
    {
        foreach ($this->headers as $header) {
            if (! $request->headers->has($header)) {
                return false;
            }
        }

        return true;
    }
}
```

This class will fail validation if any of the headers we pass in to the constructor are not present on the incoming request. This means that the request will only be validated as an API request if all of the headers are defined.

We can now add this validator to the array of validators. We have a couple options available to us, replacing, merging, or simply extending.

In our case we can simply extend to add our validator. We can do this in the `boot` method of our provider.

```php
$this->app['Dingo\Api\Http\RequestValidator']->extend(new MyApp\Http\Validation\Headers(['x-api']));
```
