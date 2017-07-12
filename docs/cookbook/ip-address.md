---
title: Retrieving IP address
---

The best way to retrieve the current IP address of the client is via middleware using
a component such as [rka-ip-address-middleware](https://github.com/akrabat/rka-ip-address-middleware).

This component can be installed via composer:

```
composer require akrabat/rka-ip-address-middleware
```

To use it, register the middleware with the `config/MiddlewareConfig.php`, providing a list
of trusted proxies (e.g. varnish servers) if you are using them:
Example
```
$checkProxyHeaders = true;
$trustedProxies = ['10.0.0.1', '10.0.0.2'];
$app->add(new RKA\Middleware\IpAddress($checkProxyHeaders, $trustedProxies));

```

Then in your `routes/routes.php` , you can use it like this
```
$app->get('/', function ($request, $response, $args) {
    $ipAddress = $request->getAttribute('ip_address');

    return $response;
});
```

The middleware stores the client's IP address in a request attribute, so access
is via `$request->getAttribute('ip_address')`.
