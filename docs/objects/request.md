---
title: Request
---

Your RawPHP's app's routes and middleware are given a PSR 7 request object that
represents the current HTTP request received by your web server. The request
object implements the [PSR 7 ServerRequestInterface][psr7] with which you can
inspect and manipulate the HTTP request method, headers, and body.

[psr7]: http://www.php-fig.org/psr/psr-7/#3-2-1-psr-http-message-serverrequestinterface

## How to get the Request object

The PSR 7 request object is injected into your RawPHP application routes as the
first argument to the route callback like this:
Inject PSR 7 request into application route callback

In `routes/routes.php`
```
<?php
$app->get('/foo', function (ServerRequestInterface $request, ResponseInterface $response) {
    // Use the PSR 7 $request object

    return $response;
});

```

The PSR 7 request object is injected into your RawPHP application _middleware_
as the first argument of the middleware callable like this:
 Inject PSR 7 request into application middleware
 
```
<?php
$app->add(function (ServerRequestInterface $request, ResponseInterface $response, callable $next) {
    // Use the PSR 7 $request object

    return $next($request, $response);
});
```

## The Request Method

Every HTTP request has a method that is typically one of:

* GET
* POST
* PUT
* DELETE
* HEAD
* PATCH
* OPTIONS

You can inspect the HTTP request's method with the Request object method
appropriately named `getMethod()`.

```
$method = $request->getMethod();
```

Because this is a common task, RawPHP's
built-in PSR 7 implementation also provides these proprietary methods that return
`true` or `false`.

* `$request->isGet()`
* `$request->isPost()`
* `$request->isPut()`
* `$request->isDelete()`
* `$request->isHead()`
* `$request->isPatch()`
* `$request->isOptions()`

It is possible to fake or _override_ the HTTP request method. This is
useful if, for example, you need to mimic a `PUT` request using a traditional
web browser that only supports `GET` or `POST` requests.

There are two ways to override the HTTP request method. You can include a
`_METHOD` parameter in a `POST` request's body. The HTTP request must use the
`application/x-www-form-urlencoded` content type.

Override HTTP method with _METHOD parameter.
```
POST /path HTTP/1.1
Host: example.com
Content-type: application/x-www-form-urlencoded
Content-length: 22

data=value&_METHOD=PUT
```
You can also override the HTTP request method with a custom
`X-Http-Method-Override` HTTP request header. This works with any HTTP request
content type.

 Override HTTP method with X-Http-Method-Override header
```
POST /path HTTP/1.1
Host: example.com
Content-type: application/json
Content-length: 16
X-Http-Method-Override: PUT
```


You can fetch the _original_ (non-overridden) HTTP method with the PSR 7 Request
object's method named `getOriginalMethod()`.

## The Request URI

Every HTTP request has a URI that identifies the requested application
resource. The HTTP request URI has several parts:

* Scheme (e.g. `http` or `https`)
* Host (e.g. `example.com`)
* Port (e.g. `80` or `443`)
* Path (e.g. `/users/1`)
* Query string (e.g. `sort=created&dir=asc`)

You can fetch the PSR 7 Request object's [URI object][psr7_uri] with its `getUri()` method:

[psr7_uri]: http://www.php-fig.org/psr/psr-7/#3-5-psr-http-message-uriinterface

```
$uri = $request->getUri();
```

The PSR 7 Request object's URI is itself an object that provides the following
methods to inspect the HTTP request's URL parts:

* `getScheme()`
* `getAuthority()`
* `getUserInfo()`
* `getHost()`
* `getPort()`
* `getPath()`
* `getBasePath()`
* `getQuery()` <small>(returns the full query string, e.g. `a=1&b=2`)</small>
* `getFragment()`
* `getBaseUrl()`

You can get the query parameters as an associative array on the Request object using `getQueryParams()`.

You can also get a single query parameter value, with optional default value if the parameter is missing, using `getQueryParam($key, $default = null)`.

<div class="alert alert-info">
    <div><strong>Base Path</strong></div>
    If your RawPHP application's front-controller lives in a physical subdirectory
    beneath your document root directory, you can fetch the HTTP request's physical
    base path (relative to the document root) with the Uri object's <code>getBasePath()</code>
    method. This will be an empty string if the RawPHP application is installed
    in the document root's top-most directory.
</div>

## The Request Headers

Every HTTP request has headers. These are metadata that describe the HTTP
request but are not visible in the request's body. RawPHP's PSR 7
Request object provides several methods to inspect its headers.

### Get All Headers

You can fetch all HTTP request headers as an associative array with the PSR 7
Request object's `getHeaders()` method. The resultant associative array's keys
are the header names and its values are themselves a numeric array of string
values for their respective header name.

Fetch and iterate all HTTP request headers as an associative array.
```
$headers = $request->getHeaders();
foreach ($headers as $name => $values) {
    echo $name . ": " . implode(", ", $values);
}
```

### Get One Header

You can get a single header's value(s) with the PSR 7 Request object's `getHeader($name)` method. This returns an array of values for the given header name. Remember, _a single
HTTP header may have more than one value!_

Get values for a specific HTTP header
```
$headerValueArray = $request->getHeader('Accept');
```
You may also fetch a comma-separated string with all values for a given header
with the PSR 7 Request object's `getHeaderLine($name)` method. Unlike the
`getHeader($name)` method, this method returns a comma-separated string.

Get single header's values as comma-separated string
```
$headerValueString = $request->getHeaderLine('Accept');
{% endhighlight %}
```

### Detect Header

You can test for the presence of a header with the PSR 7 Request object's
`hasHeader($name)` method.

Detect presence of a specific HTTP request header
```
if ($request->hasHeader('Accept')) {
    // Do something
}
```
## The Request Body

Every HTTP request has a body. If you are building a RawPHP application that
consumes JSON or XML data, you can use the PSR 7 Request object's
`getParsedBody()` method to parse the HTTP request body into a native PHP format.
RawPHP can parse JSON, XML, and URL-encoded data out of the box.

Parse HTTP request body into native PHP format
```
$parsedBody = $request->getParsedBody();
```

* JSON requests are converted into associative arrays with `json_decode($input, true)`.
* XML requests are converted into a `SimpleXMLElement` with `simplexml_load_string($input)`.
* URL-encoded requests are converted into a PHP array with `parse_str($input)`.

For URL-encoded requests, you can also get a single parameter value, with optional default value if the parameter is missing, using `getParsedBodyParam($key, $default = null)`.

Technically speaking, RawPHP's PSR 7 Request object represents the HTTP request
body as an instance of `\Psr\Http\Message\StreamInterface`. You can get
the HTTP request body `StreamInterface` instance with the PSR 7 Request object's
`getBody()` method. The `getBody()` method is preferable if the incoming HTTP
request size is unknown or too large for available memory.

Get HTTP request body
```
$body = $request->getBody();
```

The resultant `\Psr\Http\Message\StreamInterface` instance provides the following
methods to read and iterate its underlying PHP `resource`.

* `getSize()`
* `tell()`
* `eof()`
* `isSeekable()`
* `seek()`
* `rewind()`
* `isWritable()`
* `write($string)`
* `isReadable()`
* `read($length)`
* `getContents()`
* `getMetadata($key = null)`

### Reparsing the body

When calling `getParsedBody` on the Request object multiple times, the body is
only parsed once, even if the Request body is modified in the meantime.

To ensure the body is reparsed, the Request object's method `reparseBody` can be
used.

## Uploaded Files

The file uploads in `$_FILES` are available from the Request object's
`getUploadedFiles()` method. This returns an array keyed by the name of the
`<input>` element.

Get uploaded files
```
$files = $request->getUploadedFiles();
```

Each object in the `$files` array is a instance of
`\Psr\Http\Message\UploadedFileInterface` and supports the following methods:

* `getStream()`
* `moveTo($targetPath)`
* `getSize()`
* `getError()`
* `getClientFilename()`
* `getClientMediaType()`

See the [cookbook](/docs/cookbook/uploading-files.html) on how to upload files using a POST form.

## Request Helpers

RawPHP's PSR 7 Request implementation provides these additional proprietary methods
to help you further inspect the HTTP request.

### Detect XHR requests

You can detect XHR requests with the Request object's `isXhr()` method. This
method detects the presence of the `X-Requested-With` HTTP request header and
ensures its value is `XMLHttpRequest`.

 Example XHR request.
```
POST /path HTTP/1.1
Host: example.com
Content-type: application/x-www-form-urlencoded
Content-length: 7
X-Requested-With: XMLHttpRequest

foo=bar
```

```
if ($request->isXhr()) {
    // Do something
}
```

### Content Type

You can fetch the HTTP request content type with the Request object's `getContentType()` method. This returns the `Content-Type` header's full value as provided by the HTTP client.

```
$contentType = $request->getContentType();
```

### Media Type

You may not want the complete `Content-Type` header. What if, instead, you only want the media type? You can fetch the HTTP request media type with the Request object's `getMediaType()` method.

```
$mediaType = $request->getMediaType();
```

You can fetch the appended media type parameters as an associative array with the Request object's `getMediaTypeParams()` method.

```
$mediaParams = $request->getMediaTypeParams();
```

### Character Set

One of the most common media type parameters is the HTTP request character set. The Request object provides a dedicated method to retrieve this media type parameter.

```
$charset = $request->getContentCharset();
```

### Content Length

You can fetch the HTTP request content length with the Request object's `getContentLength()` method.

```
$length = $request->getContentLength();
```

### Request Parameter

To fetch single request parameter value, use methods: `getParam()`, `getQueryParam()`, `getParsedBodyParam()`, `getCookieParam()`, `getServerParam()`, counterparts of PSR-7's plural form get*Params() methods.

For example, to get a single Server Parameter:

```
$foo = $request->getServerParam('HTTP_NOT_EXIST', 'default_value_here');
```

## Route Object

Sometimes in middleware you require the parameter of your route.

In this example we are checking first that the user is logged in and second that the user has permissions to view the particular video they are attempting to view.

```
    $app->get('/course/{id}', Video::class.":watch")->add(Permission::class)->add(Auth::class);

    //.. In the Permission Class's Invoke
    /** @var $route
    $route = $request->getAttribute('route');
    $courseId = $route->getArgument('id');
```

## Media Type Parsers

RawPHP looks as the request's media type and if it recognises it, will parse it into structured data available via ``$request->getParsedBody()``. This is usually an array, but is an object for XML media types.

The following media types are recognised and parsed:

* application/x-www-form-urlencoded
* application/json
* application/xml & text/xml

If you want RawPHP to parse content from a different media type then you need to either parse the raw body yourself or register a new media parser. Media parsers are simply callables that accept an ``$input`` string and return a parsed object or array.

Register a new media parser in an application or route middleware. Note that you must register the parser before you try to access the parsed body for the first time.

For example, to automatically parse JSON that is sent with a ``text/javascript`` content type, you register a media type parser in middleware like this:

```
// Add the middleware
$app->add(function ($request, $response, $next) {
    // add media parser
    $request->registerMediaTypeParser(
        "text/javascript",
        function ($input) {
            return json_decode($input, true);
        }
    );

    return $next($request, $response);
});
```

## Attributes

With PSR-7 it is possible to inject objects/values into the request object for further processing. In your applications middleware often need to pass along information to your route closure and the way to do is it is to add it to the request object via an attribute.

Example, Setting a value on your request object.

```
$app->add(function ($request, $response, $next) {
    $request = $request->withAttribute('session', $_SESSION); //add the session storage to your request as [READ-ONLY]
    return $next($request, $response);
});
```


Example, how to retrieve the value.

```
$app->get('/test', function ($request, $response, $args) {
    $session = $request->getAttribute('session'); //get the session from the request

    return $response->write('Yay, ' . $session['name']);
});
```

The request object also has bulk functions as well. `$request->getAttributes()` and `$request->withAttributes()`
