---
title: Router
---

## How to create routes

You can define application routes using proxy methods on the `RawPHP` instance. The RawPHP Framework provides methods for the most popular HTTP methods.

### GET Route

You can add a route that handles only `GET` HTTP requests with the RawPHP
application's `get()` method. It accepts two arguments:

1. The route pattern (with optional named placeholders)
2. The route callback

```
$app->get('/books/{id}', function ($request, $response, $args) {
    // Show book identified by $args['id']
});
```

### POST Route

You can add a route that handles only `POST` HTTP requests with the RawPHP
application's `post()` method. It accepts two arguments:

1. The route pattern (with optional named placeholders)
2. The route callback

```
$app->post('/books', function ($request, $response, $args) {
    // Create new book
});
```

### PUT Route

You can add a route that handles only `PUT` HTTP requests with the RawPHP
application's `put()` method. It accepts two arguments:

1. The route pattern (with optional named placeholders)
2. The route callback

```
$app->put('/books/{id}', function ($request, $response, $args) {
    // Update book identified by $args['id']
});
```
### DELETE Route

You can add a route that handles only `DELETE` HTTP requests with the RawPHP
application's `delete()` method. It accepts two arguments:

1. The route pattern (with optional named placeholders)
2. The route callback

```
$app->delete('/books/{id}', function ($request, $response, $args) {
    // Delete book identified by $args['id']
});
```

### OPTIONS Route

You can add a route that handles only `OPTIONS` HTTP requests with the RawPHP
application's `options()` method. It accepts two arguments:

1. The route pattern (with optional named placeholders)
2. The route callback

```
$app->options('/books/{id}', function ($request, $response, $args) {
    // Return response headers
});
```

### PATCH Route

You can add a route that handles only `PATCH` HTTP requests with the RawPHP
application's `patch()` method. It accepts two arguments:

1. The route pattern (with optional named placeholders)
2. The route callback

```
$app->patch('/books/{id}', function ($request, $response, $args) {
    // Apply changes to book identified by $args['id']
});
```

### Any Route

You can add a route that handles all HTTP request methods with the RawPHP application's `any()` method. It accepts two arguments:

1. The route pattern (with optional named placeholders)
2. The route callback

```
$app->any('/books/[{id}]', function ($request, $response, $args) {
    // Apply changes to books or book identified by $args['id'] if specified.
    // To check which method is used: $request->getMethod();
});
```

Note that the second parameter is a callback. You could specify a Class (which need a `__invoke()` implementation) instead of a Closure. You can then do the mapping somewhere else:

```
$app->any('/user', 'MyRestfulController');
```

### Custom Route

You can add a route that handles multiple HTTP request methods with the RawPHP application's `map()` method. It accepts three arguments:

1. Array of HTTP methods
2. The route pattern (with optional named placeholders)
3. The route callback

```
$app->map(['GET', 'POST'], '/books', function ($request, $response, $args) {
    // Create new book or list all books
});
```
## Route callbacks

Each routing method described above accepts a callback routine as its final argument. This argument can be any PHP callable, and by default it accepts three arguments.

**Request**
: The first argument is a `Psr\Http\Message\ServerRequestInterface` object that represents the current HTTP request.

**Response**
: The second argument is a `Psr\Http\Message\ResponseInterface` object that represents the current HTTP response.

**Arguments**
: The third argument is an associative array that contains values for the current route's named placeholders.

### Writing content to the response

There are two ways you can write content to the HTTP response. First, you can simply `echo()` content from the route callback. This content will be appended to the current HTTP response object. Second, you can return a `Psr\Http\Message\ResponseInterface` object.

### Closure binding

If you use a `Closure` instance as the route callback, the closure's state is bound to the `Container` instance. This means you will have access to the DI container instance _inside_ of the Closure via the `$this` keyword:

```
$app->get('/hello/{name}', function ($request, $response, $args) {
    // Use app HTTP cookie service
    $this->get('cookies')->set('name', [
        'value' => $args['name'],
        'expires' => '7 days'
    ]);
});
```

## Route placeholders

Each routing method described above accepts a URL pattern that is matched against the current HTTP request URI. Route patterns may use named placeholders to dynamically match HTTP request URI segments.

### Format

A route pattern placeholder starts with a `{`, followed by the placeholder name, ending with a `}`. This is an example placeholder named `name`:

```
$app->get('/hello/{name}', function ($request, $response, $args) {
    echo "Hello, " . $args['name'];
});
```

### Optional segments

To make a section optional, simply wrap in square brackets:

```
$app->get('/users[/{id}]', function ($request, $response, $args) {
    // responds to both `/users` and `/users/123`
    // but not to `/users/`
});
```


Multiple optional parameters are supported by nesting:

```
$app->get('/news[/{year}[/{month}]]', function ($request, $response, $args) {
    // reponds to `/news`, `/news/2016` and `/news/2016/03`
});
```

For "Unlimited" optional parameters, you can do this:

```
$app->get('/news[/{params:.*}]', function ($request, $response, $args) {
    $params = explode('/', $request->getAttribute('params'));

    // $params is an array of all the optional segments
});
```

In this example, a URI of `/news/2016/03/20` would result in the `$params` array
containing three elements: `['2016', '03', '20']`.


### Regular expression matching

By default the placeholders are written inside `{}` and can accept any
values. However, placeholders can also require the HTTP request URI to match a particular regular expression. If the current HTTP request URI does not match a placeholder regular expression, the route is not invoked. This is an example placeholder named `id` that requires one or more digits.

```
$app->get('/users/{id:[0-9]+}', function ($request, $response, $args) {
    // Find user identified by $args['id']
});
```

## Route names

Application routes can be assigned a name. This is useful if you want to programmatically generate a URL to a specific route with the router's `pathFor()` method. Each routing method described above returns a route object exposes a `setName()` method.

```
$app->get('/hello/{name}', function ($request, $response, $args) {
    echo "Hello, " . $args['name'];
})->setName('hello');
```

You can generate a URL for this named route with the application router's `pathFor()`  method.

```
echo $app->getContainer()->get('router')->pathFor('hello', [
    'name' => 'Josh'
]);
// Outputs "/hello/Josh"
```

The router's `pathFor()` method accepts two arguments:

1. The route name
2. Associative array of route pattern placeholders and replacement values

## Route groups

To help organize routes into logical groups, the RawPHP also provides a `group()` method. Each group's route pattern is prepended to the routes or groups contained within it, and any placeholder arguments in the group pattern are ultimately made available to the nested routes:

```
$app->group('/users/{id:[0-9]+}', function () {
    $this->map(['GET', 'DELETE', 'PATCH', 'PUT'], '', function ($request, $response, $args) {
        // Find, delete, patch or replace user identified by $args['id']
    })->setName('user');
    $this->get('/reset-password', function ($request, $response, $args) {
        // Route for /users/{id:[0-9]+}/reset-password
        // Reset the password for user identified by $args['id']
    })->setName('user-password-reset');
});
```

Note inside the group closure, `$this` is used instead of `$app`. RawPHP binds the closure to the application instance for you, just like it is the case with route callback binds with container instance.

* inside group closure, `$this` is bound to the instance of `\App`
* inside route closure, `$this` is bound to the instance of `\Container`

## Route middleware

You can also attach middleware to any route or route group. [Learn more](/docs/concepts/middleware.html).

## Router caching

It's possible to enable router cache by setting valid filename in default RawPHP settings. [Learn more](/docs/objects/application.html#rawphp-default-settings).

## Container Resolution

You are not limited to defining a function for your routes. In RawPHP there are a few different ways to define your route action functions.

In addition to a function, you may use:

 - `container_key:method`
 - `Class:method`
 - An invokable class
 - `container_key`
 
This functionality is enabled by RawPHP's Callable Resolver Class. It translates a string entry into a function call.
Example:

```
$app->get('/', '\HomeController:home');
```

Alternatively, you can take advantage of PHP's `::class` operator which works well with IDE lookup systems and produces the same result: 

```
$app->get('/', \HomeController::class . ':home');
```

In this code above we are defining a `/` route and telling RawPHP to execute the `home()` method on the `HomeController` class.

RawPHP first looks for an entry of `HomeController` in the container, if it's found it will use that instance otherwise it will call it's constructor with the container as the first argument. Once an instance of the class is created it will then call the specified method using whatever Strategy you have defined.

### Registering a controller with the container

Create a controller with the `home` action method. The constructor should accept
the dependencies that are required. For example:

```
class HomeController
{
    protected $view;
    
    public function __construct(\Slim\Views\Twig $view) {
        $this->view = $view;
    }
    public function home($request, $response, $args) {
      // your code here
      // use $this->view to render the HTML
      return $response;
    }
}
```

Create a factory in the container that instantiates the controller with the dependencies:

In `config/ControllerConfig.php`
```
$container['HomeController'] = function($container) {
    $view = $c->get("view"); // retrieve the 'view' from the container
    return new HomeController($view);
};
```

This allows you to leverage the container for dependency injection and so you can 
inject specific dependencies into the controller.


### Allow RawPHP to instantiate the controller

Alternatively, if the class does not have an entry in the container, then RawPHP
will pass the container's instance to the constructor. You can construct controllers 
with many actions instead of an invokable class which only handles one action.

```
class HomeController 
{
   protected $container;

   // constructor receives container instance
   public function __construct(ContainerInterface $container) {
       $this->container = $container;
   }
   
   public function home($request, $response, $args) {
        // your code
        // to access items in the container... $this->container->get('');
        return $response;
   }
   
   public function contact($request, $response, $args) {
        // your code
        // to access items in the container... $this->container->get('');
        return $response;
   }
}
```

You can use your controller methods like so.

```
$app->get('/', \HomeController::class . ':home');
$app->get('/contact', \HomeController::class . ':contact');
```

### Using an invokable class

You do not have to specify a method in your route callable and can just set it to be an invokable class such as:

```
class HomeAction
{
   protected $container;
   
   public function __construct(ContainerInterface $container) {
       $this->container = $container;
   }
   
   public function __invoke($request, $response, $args) {
        // your code
        // to access items in the container... $this->container->get('');
        return $response;
   }
}
```

You can use this class like so.

```
$app->get('/', \HomeAction::class);
```

Again, as with controllers, if you register the class name with the container, then you 
can create a factory and inject just the specific dependencies that you require into your
action class.
