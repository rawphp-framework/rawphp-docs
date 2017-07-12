---
title: Setting up CORS in RawPHP
---

CORS - Cross origin resource sharing

A good flowchart for implementing CORS support Reference: 

[CORS server flowchart](http://www.html5rocks.com/static/images/cors_server_flowchart.png)

You can test your CORS Support here: http://www.test-cors.org/

You can read the specification here: https://www.w3.org/TR/cors/


## The simple solution

For simple CORS requests, the server only needs to add the following header to its response:

`Access-Control-Allow-Origin: <domain>, ... | *`

The following code should enable lazy CORS.

In your `routes/routes.php` file, you can do the below directly but it isnt advisable
```
$app->options('/{routes:.+}', function ($request, $response, $args) {
    return $response;
});

$app->add(function ($req, $res, $next) {
    $response = $next($req, $res);
    return $response
            ->withHeader('Access-Control-Allow-Origin', 'http://mysite')
            ->withHeader('Access-Control-Allow-Headers', 'X-Requested-With, Content-Type, Accept, Origin, Authorization')
            ->withHeader('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, OPTIONS');
});

```
A better and neater way would be to 
* Create a Middleware file for it in `app/Middlewares`
* Then add it to the list in `config/MiddlewareConfig`

## Create a Middleware file for it in `app/Middlewares`


```
<?php 

namespace App\Middlewares;

class CorsMiddleware extends Middleware{
	
public function __invoke($req, $res, $next) {
    $response = $next($req, $res);
    return $response
            ->withHeader('Access-Control-Allow-Origin', 'http://mysite')
            ->withHeader('Access-Control-Allow-Headers', 'X-Requested-With, Content-Type, Accept, Origin, Authorization')
            ->withHeader('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, OPTIONS');
}

}
```

## Then add it to the list in `config/MiddlewareConfig.php`
```
$app->add(new \App\Middlewares\CorsMiddleware($container));
```

## Access-Control-Allow-Methods

The following middleware can be used to query RawPHP's router and get a list of methods a particular pattern implements.

Here is a complete example application:

In `config/DatabaseConfig.php`

```
// Add this RawPHP setting is required for the middleware to work
$app = new Slim\App([
    "settings"  => [
        'displayErrorDetails' => true,
        "determineRouteBeforeAppMiddleware" => true, // add this line
    ]
]);
```


## Cors middleware 
// This is the middleware
// It will add the Access-Control-Allow-Methods header to every request

```
<?php 

namespace App\Middlewares;

class CorsMiddleware extends Middleware{
	
public function __invoke($request, $response, $next) {
    $route = $request->getAttribute("route");

    $methods = [];

    if (!empty($route)) {
        $pattern = $route->getPattern();

        foreach ($this->router->getRoutes() as $route) {
            if ($pattern === $route->getPattern()) {
                $methods = array_merge_recursive($methods, $route->getMethods());
            }
        }
        //Methods holds all of the HTTP Verbs that a particular route handles.
    } else {
        $methods[] = $request->getMethod();
    }
    
    $response = $next($request, $response);


    return $response->withHeader("Access-Control-Allow-Methods", implode(",", $methods));
}
}

```

## Every Route now has Cors
Example, in your `routes/routes.php` file 
```
$app->get("/api/{id}", function($request, $response, $arguments) {
});

$app->post("/api/{id}", function($request, $response, $arguments) {
});

$app->map(["DELETE", "PATCH"], "/api/{id}", function($request, $response, $arguments) {
});
```
// Pay attention to this when you are using some javascript front-end framework and you are using groups in slim php
$app->group('/api', function () {
    // Due to the behaviour of browsers when sending PUT or DELETE request, you must add the OPTIONS method. Read about preflight.
    $this->map(['PUT', 'OPTIONS'], '/{user_id:[0-9]+}', function ($request, $response, $arguments) {
        // Your code here...
    });
});

```


