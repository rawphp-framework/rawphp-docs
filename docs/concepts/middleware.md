---
title: Middleware
---

You can write a certain code that will only run _before_ and _after_ your RawPHP application to manipulate the
Request and Response objects as you see fit. This is called _middleware_.
There are many reasons to do this, some are:
* Protecting your site from cross-site request forgery. 
* Authenticating requests before your app runs. 

Middleware is perfect for these kind of scenarios where you want a certain piece of code to run before any request is processed in your platform.

## What is middleware?

Technically speaking, a middleware is a _callable_ that accepts three arguments:

1. `\Psr\Http\Message\ServerRequestInterface` - The PSR7 request object
2. `\Psr\Http\Message\ResponseInterface` - The PSR7 response object
3. `callable` - The next middleware callable

It can do whatever is appropriate with these objects. The only hard requirement
is that a middleware **MUST** return an instance of `\Psr\Http\Message\ResponseInterface`.
Each middleware **SHOULD** invoke the next middleware and pass it Request and
Response objects as arguments.

## How does middleware work?

RawPHP adds middleware as concentric layers surrounding your core application. Each new middleware layer surrounds
any existing middleware layers. The concentric structure expands outwardly as additional middleware layers are added.

The execution order is last-in-first-out, that is, the last middleware layer added is the first to be executed.

When you run the RawPHP application, the Request and Response objects traverse the middleware structure from the outside in. They first enter the outer-most middleware, then the next outer-most middleware, (and so on), until they ultimately arrive at the RawPHP application itself. After the RawPHP application dispatches the appropriate route, the resultant Response object exits the RawPHP application and traverses the middleware structure from the inside out. Ultimately, a final
Response object exits the outer-most middleware, is serialized into a raw HTTP response, and is returned to the HTTP client. Here's a diagram that illustrates
the middleware process flow:

<div style="padding: 2em 0; text-align: center">
    <img src="/docs/images/middleware.png" alt="Middleware architecture" style="max-width: 80%;"/>
</div>

## How do I write middleware?

There are a few easy steps to create a middleware

* Create the middleware file  
* Define and Register the Middleware
* Create a shorthand for it (Recommended but optional)
* Attach it to routes, route groups or the entire application

## How to create a middleware file
You can create middlewares by creating the file in `App\Middlewares\`. The name must end with `Middleware` as in the examples below. The contents of the file will look like the `AuthMiddleware` Example below. 

Middleware is a callable that accepts three arguments: a Request object, a Response object, and the next middleware. Each middleware **MUST** return an instance of `\Psr\Http\Message\ResponseInterface`.
RawPHP provides some middlewares out of the box, they are all located in `App\Middlewares\`
```
AuthMiddleware
CsrfViewMiddleware
GuestMiddleware
OldInputMiddleware
ValidationMiddleware
```
All middlewares must :
* Have the `namespace App\Middlewares`; 
* Extend the parent class `Middleware`.
* Have at least an `__invoke()` method with `$request, $response and $next` as its parameters
* Return a `$next($request, $response)`

Below is an example of the AuthMiddleware that comes default with RawPHP.

```
<?php 

namespace App\Middlewares; 

class AuthMiddleware extends Middleware{
	
	/**
	* Get the CSRF token, pass it to the view then move unto the next middleware
	* this can be access from the view with {{ csrf.field | raw }}
	* @return $response
	*/
	public function __invoke($request, $response, $next){
		
		if(!$this->container->auth->check()){
			$this->container->flash->addMessage('error', 'You have to be logged in to continue');
			
			//redirect them to login page
			return $response->withRedirect($this->container->router->pathFor('auth.sign'));
		} 
		 //this can be access from the view with {{ csrf.field | raw }}
		 
		 
		//the below must be done in all middlewares
		$response = $next($request, $response);
		
		return $response;
	}
}

```
## Define the Middleware
There are 2 ways to define your middleware after creating it. 
* Add it to the list of middleware definitions in `config/MiddlewareConfig.php`
* Import it at the top of your `routes/routes.php`

## Create a shorthand for it (Recommended but optional)
You can create a short hand name for your new middleware by adding it to the list of shorthand definitions in `config/ContainerConfig.php`

## How to use your middleware
Finally, you can attach it to routes and route groups in `routes/routes.php`. In the example below, we will be importing the `AuthMiddleware` we created earlier and will be attaching it to a specific route.

`routes/routes.php`

```
<?php
//import the AuthMiddleware (if you havent already imported it in config/MiddlewareConfig.php
use App\Middlewares\AuthMiddleware;

//attach it to a route by appending ->add(new AuthMiddleware($container)); to the route definition

$app->get('/auth/signup', 'AuthController:getSignup')->setName('auth.signup')->add(new AuthMiddleware($container));


//Or attach it to a route group using the same method
$app->group('', function ()  use ($app) {
$app->get('/auth/signup', 'AuthController:getSignup')->setName('auth.signup');
$app->post('/auth/signup', 'AuthController:postSignup');

$app->get('/auth/signin', 'AuthController:getSignin')->setName('auth.signin');
$app->post('/auth/signin', 'AuthController:postSignin');

$app->get('/auth/password/forgot', 'AuthController:forgotPassword')->setName('auth.password.forgot');
$app->post('/auth/password/forgot', 'AuthController:forgotPassword');

})->add(new AuthMiddleware($container));

```
# More on middlewares

Middleware is a callable that accepts three arguments: a Request object, a Response object, and the next middleware. Each middleware **MUST** return an instance of `\Psr\Http\Message\ResponseInterface`.

### Closure middleware example.

This example middleware is a Closure.

```
<?php
/**
 * Example middleware closure
 *
 * @param  \Psr\Http\Message\ServerRequestInterface $request  PSR7 request
 * @param  \Psr\Http\Message\ResponseInterface      $response PSR7 response
 * @param  callable                                 $next     Next middleware
 *
 * @return \Psr\Http\Message\ResponseInterface
 */
function ($request, $response, $next) {
    $response->getBody()->write('BEFORE');
    $response = $next($request, $response);
    $response->getBody()->write('AFTER');

    return $response;
};
```

### Invokable class middleware example

This example middleware is an invokable class that implements the magic `__invoke()` method.

```
<?php
class ExampleMiddleware
{
    /**
     * Example middleware invokable class
     *
     * @param  \Psr\Http\Message\ServerRequestInterface $request  PSR7 request
     * @param  \Psr\Http\Message\ResponseInterface      $response PSR7 response
     * @param  callable                                 $next     Next middleware
     *
     * @return \Psr\Http\Message\ResponseInterface
     */
    public function __invoke($request, $response, $next)
    {
        $response->getBody()->write('BEFORE');
        $response = $next($request, $response);
        $response->getBody()->write('AFTER');

        return $response;
    }
}
```

To use this class as a middleware, you can use `->add( new ExampleMiddleware() );` function chain after the `$app`, `Route`,  or `group()`, which in the code below, any one of these, could represent $subject.

```
$subject->add( new ExampleMiddleware() );
```

## How do I add middleware?

You may add middleware to a RawPHP application, to an individual RawPHP application route or to a route group. All scenarios accept the same middleware and implement the same middleware interface.

### Application middleware

Application middleware is invoked for every *incoming* HTTP request. Add application middleware with the Slim application instance's `add()` method. This example adds the Closure middleware example above:

```
<?php

$app->add(function ($request, $response, $next) {
	$response->getBody()->write('BEFORE');
	$response = $next($request, $response);
	$response->getBody()->write('AFTER');

	return $response;
});
//
$app->get('/', function ($request, $response, $args) {
	$response->getBody()->write(' Hello ');

	return $response;
});

```

This would output this HTTP response body:

    BEFORE Hello AFTER

### Route middleware

Route middleware is invoked _only if_ its route matches the current HTTP request method and URI. Route middleware is specified immediately after you invoke any of the Slim application's routing methods (e.g., `get()` or `post()`). Each routing method returns an instance of `\Slim\Route`, and this class provides the same middleware interface as the Slim application instance. Add middleware to a Route with the Route instance's `add()` method. This example adds the Closure middleware example above:

```
<?php

$mw = function ($request, $response, $next) {
    $response->getBody()->write('BEFORE');
    $response = $next($request, $response);
    $response->getBody()->write('AFTER');

    return $response;
};

$app->get('/', function ($request, $response, $args) {
	$response->getBody()->write(' Hello ');

	return $response;
})->add($mw);

```

This would output this HTTP response body:

    BEFORE Hello AFTER

### Group Middleware

In addition to the overall application, and standard routes being able to accept middleware, the `group()` multi-route definition functionality, also allows individual routes internally. Route group middleware is invoked _only if_ its route matches one of the defined HTTP request methods and URIs from the group. To add middleware within the callback, and entire-group middleware to be set by chaining `add()` after the `group()` method.

Sample Application, making use of callback middleware on a group of url-handlers
```
<?php

require_once __DIR__.'/vendor/autoload.php';

$app = new \Slim\App();

$app->get('/', function ($request, $response) {
    return $response->getBody()->write('Hello World');
});

$app->group('/utils', function () use ($app) {
    $app->get('/date', function ($request, $response) {
        return $response->getBody()->write(date('Y-m-d H:i:s'));
    });
    $app->get('/time', function ($request, $response) {
        return $response->getBody()->write(time());
    });
})->add(function ($request, $response, $next) {
    $response->getBody()->write('It is now ');
    $response = $next($request, $response);
    $response->getBody()->write('. Enjoy!');

    return $response;
});
```

When calling the `/utils/date` method, this would output a string similar to the below

    It is now 2015-07-06 03:11:01. Enjoy!

visiting `/utils/time` would output a string similar to the below

    It is now 1436148762. Enjoy!

but visiting `/` *(domain-root)*, would be expected to generate the following output as no middleware has been assigned

    Hello World

### Passing variables from middleware
The easiest way to pass attributes from middleware is to use the request's
attributes.

Setting the variable in the middleware:

```
$request = $request->withAttribute('foo', 'bar');
```

Getting the variable in the route callback:
```
$foo = $request->getAttribute('foo');
```
