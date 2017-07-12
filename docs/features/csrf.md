---
title: CSRF Protection
---
CSRF stands for Cross-site request forgery.It is a type of malicious exploit whereby unauthorized commands are performed on behalf of an authenticated user. 
This component generates a unique token per request that validates subsequent POST requests from client-side HTML forms.

## Installation
It is already installed as a Middleware by default in `app/Middlewares/CsrfViewMiddleware.php`

## Usage
All your forms in RawPHP must contain an extra `{{ csrf.field | raw }}`. 
You need to add that anywhere inbetween the opening and closing form tags, like this:

```
<form method="post" action"/whatever-route-url">

<!-- Your other form code here -->
  
  {{ csrf.field | raw }}
</form>
```

It will be stored as a hidden field within the form field. Which means, if you inspect element or view source of the form on your browser, you will see the CSRF hidden fields.

The above is enough to get it working. 

## How to Fetch the CSRF token name and value
You may never need to use this, but if you do, you can retrive the latest CSRF token's name and value are available as attributes on the
PSR7 request object. The CSRF token name and value are unique for each request.
You can fetch the current CSRF token name and value  from your controller or `routes/routes.php` like this.

```
$app->get('/foo', function ($req, $res, $args) {
    // CSRF token name and value
    $nameKey = $this->csrf->getTokenNameKey();
    $valueKey = $this->csrf->getTokenValueKey();
    $name = $req->getAttribute($nameKey);
    $value = $req->getAttribute($valueKey);

    // Render HTML form which POSTs to /bar with two hidden input fields for the
    // name and value:
    // <input type="hidden" name="<?= $nameKey ?>" value="<?= $name ?>">
    // <input type="hidden" name="<?= $valueKey ?>" value="<?= $value ?>">
});
```


If CSRF token isn't successful, your closure function won't run. You'll get a CSRF error instead.
```
$app->post('/bar', function ($req, $res, $args) {
    // CSRF protection successful if you reached
    // this far.
});
```
