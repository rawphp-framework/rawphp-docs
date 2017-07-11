---
title: Lifecycle
---

## Application Lifecycle

### Instantiation

The entry point for all requests to a RawPHP application is the `public/index.php` file. All requests are directed to this file by your web server (Apache / Nginx) configuration. The `index.php` file doesn't contain much code. Rather, it is simply a starting point for loading the rest of the framework.

The `index.php` file loads the Composer generated autoloader definition, and then retrieves an instance of the RawPHP application from `bootstrap/app.php` script. The first action taken by RawPHP itself is to create an instance of the application / [service container](di.md).

### Route Definitions
The routes file is located in `routes/route.php`. You are going to be using this file a lot.
Routes can be defined using the application instance's `get()`, `post()`, `put()`, `delete()`, `patch()`, `head()`, and `options()` routing methods. These instance methods register a route with the application's Router object. Each routing method returns the Route instance so you can immediately invoke the Route instance's methods to add middleware or assign a name.
From this routes file, you can connect to the controller or view that is meant to process the request. 

### Application Runner

Third,the application instance's is invoked in the `public/index.php` file with `$app->run()` method. This method starts the following process:

#### A. Enter Middleware Stack

The `$app->run()` method begins to inwardly traverse the application's middleware stack. This is a concentric structure of middleware layers that receive (and optionally manipulate) the Environment, Request, and Response objects before (and after) the RawPHP application runs. The RawPHP application is the inner-most layer of the concentric middleware structure. Each middleware layer is invoked inwardly beginning from the outer-most layer.

#### B. Run Application

After the `$app->run()` method reaches the inner-most middleware layer, it invokes the application instance and dispatches the current HTTP request to the appropriate application route object. If a route matches the HTTP method and URI, the route's middleware and callable are invoked. If a matching route is not found, the Not Found or Not Allowed handler is invoked.

#### D. Exit Middleware Stack

After the application dispatch process completes, each middleware layer reclaims control outwardly, beginning from the inner-most layer.

#### E. Send HTTP Response

After the outer-most middleware layer cedes control, the application instance prepares, serializes, and returns the HTTP response. The HTTP response headers are set with PHP's native `header()` method, and the HTTP response body is output to the current output buffer.
