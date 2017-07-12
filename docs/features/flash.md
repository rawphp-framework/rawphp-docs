---
title: Flash Messages
---
Flash messages are a cool way to display notifications and alerts to the user. These messages will dissapear once the page is refreshed.

## Usage
You can send flash messages in your controller methods like this:
` $this->flash->addMessage('Test', 'This is a message');`
This will display as a flash message on the view to the user. There are different colors you can use by simply changing the name passed as a parameter. Example
```
 $this->flash->addMessage('default', 'This is a message'); //plain
 $this->flash->addMessage('info', 'This is a message'); //blue
 $this->flash->addMessage('warning', 'This is a message'); //yellow
 $this->flash->addMessage('success', 'This is a message');  //green
 $this->flash->addMessage('error', 'This is a message'); //red
    

```
Below is an example usage. The user will see the message when the page refreshes or when they move to the next page
```
$app->get('/foo', function ($req, $res, $args) {
    // Set flash message for next request
    $this->flash->addMessage('info', 'This is a message');

    // Redirect
    return $res->withStatus(302)->withHeader('Location', '/bar');
});
```

You can get previous flash message
```
$app->get('/bar', function ($req, $res, $args) {
    // Get flash messages from previous request
    $messages = $this->flash->getMessages();
    print_r($messages);
});

```

Please note that a message could be a string, object or array. Please check what your storage can handle.
