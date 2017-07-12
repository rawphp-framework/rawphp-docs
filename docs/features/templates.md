---
title: Templates
---

RawPHP can work without a view layer. This is useful when using it to build an API/microservice platform. To make this possible, at the end of each method in your controller, you return json instead of returning a view.

RawPHP comes bundled with the Twig view. Twig is very much like the blade template used by laravel or cake cemplate that CakePHP uses.
You can write html , php and some short hand twig specific codes inside it.


## The Twig component

Below is an example of how to display a twig view from your controller method
```
public function getSignup($request , $response){
	
	return $this->view->render($response,'auth/signup.twig');  //this displays the twig view in resources/views/auth/signup.twig
	}
```

The [Twig-View][twigview] PHP component helps you render [Twig][twig]
templates in your application. This component is available on Packagist, and
it's easy to install with Composer like this:

[twigview]: https://github.com/slimphp/Twig-View
[twig]: http://twig.sensiolabs.org/



Note : "cache" could be set to false to disable it, see also 'auto_reload' option, useful in development environment. For more information, see [Twig environment options](http://twig.sensiolabs.org/doc/2.x/api.html#environment-options)


// Render Twig template in route
```
$app->get('/hello/{name}', function ($request, $response, $args) {
    return $this->view->render($response, 'profile.html', [
        'name' => $args['name']
    ]);
})->setName('profile');
```

### The path_for() method

The RawPHP's twig component exposes a custom `path_for()` function
to your Twig templates. You can use this function to generate complete
URLs to any named route in your Slim application. The `path_for()`
function accepts two arguments:

1. A route name
2. A hash of route placeholder names and replacement values

The second argument's keys should correspond to the selected route's pattern
placeholders. This is an example Twig template that draws a link URL
for the "profile" named route shown in the example Slim application above.


Below is a sample usage of `path_for()` to specify a route already created in `routes/routes.php` and specify `josh` as a route parameter.
```
{% extends "layout.html" %}
{% block body %}
<h1>User List</h1>
<ul>
    <li><a href="{{ path_for('profile', { 'name': 'josh' }) }}">Josh</a></li>
</ul>
{% endblock %}
```
## Other template systems

You are not limited to the `Twig-View` components. You
can install and use blade, cake template or any PHP template system provided that you ultimately write the rendered
template output to the PSR 7 Response object's body.
