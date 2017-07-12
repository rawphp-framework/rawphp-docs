---
title: 405 Not Allowed Handler
---

If your RawPHP Framework application has a route that matches the current HTTP request URI **but NOT the HTTP request method**, the application invokes its Not Allowed handler and returns a `HTTP/1.1 405 Not Allowed` response to the HTTP client.

## Default Not Allowed handler

Each RawPHP Framework application has a default Not Allowed handler. This handler sets the Response status to `405`, it sets the content type to `text/html`, it adds a `Allowed:` HTTP header with a comma-delimited list of allowed HTTP methods, and it writes a simple explanation to the Response body.

## Custom Not Allowed handler

A RawPHP Framework application's Not Allowed handler is a Pimple service. You can substitute your own Not Allowed handler by defining a custom Pimple factory method with the application container.

in your `config/ContainerConfig.php`
```
$container['notAllowedHandler'] = function ($container) {
    return function ($request, $response, $methods) use ($container) {
        return $container['response']
            ->withStatus(405)
            ->withHeader('Allow', implode(', ', $methods))
            ->withHeader('Content-type', 'text/html')
            ->write('Method must be one of: ' . implode(', ', $methods));
    };
};
```

**N.B** Also check out [Not Found](/docs/handlers/not-found.html) docs 

In this example, we define a new `notAllowedHandler` factory that returns a callable. The returned callable accepts three arguments:

1. A `\Psr\Http\Message\ServerRequestInterface` instance
2. A `\Psr\Http\Message\ResponseInterface` instance
3. A numeric array of allowed HTTP method names

The callable **MUST** return an appropriate `\Psr\Http\Message\ResponseInterface` instance.
