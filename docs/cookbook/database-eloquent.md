---
title: Using Laravel's Eloquent or CakePHP's ORM with RawPHP
---

You can use a database ORM such as [CakePHP's ORM](https://book.cakephp.org/3.0/en/orm.html) or [Eloquent](https://laravel.com/docs/5.1/eloquent) to connect your RawPHP application to a database. 
This article will teach how to set them up, for how to perform different database queries follow the link above to readup it up on their web pages depending on which of them you choose to use.

RawPHP comes with Laravel enabled by default, to use CakePHP ORM, you just have to turn it on. You can have both of them turned on at the same time, which means you can write both Cakephp and Laravel ORM in the same controller methods. Awesome, isn't it?



# Adding Laravel Eloquent and CakePHP's ORM to your application
Steps:
* Specify the database details in `config/DatabaseConfig.php`
* Enable them in  `config/DatabaseConfig.php`. Laravel is enabled by default, to enable CakePHP just uncomment it.
* Import them in the Model files you wish to use eg. `User.php`

## Configure Eloquent

Specify your database settings `config/DatabaseConfig.php` . Note that Laravel and CakePHP use different settings, to use both of them, just fill the same database details in both. This does not create the connection, so it is advised that you enter the same details in both of them even if you'll eventually use only one of them.

Laravel uses the `DB` by default, CakePHP uses `CakeDB` as the name as specified below:

`config/DatabaseConfig.php`

```
<?php 

/**
* 
* Database settings
* 
*/
$app = new \Slim\App([
	'settings' => [
		'displayErrorDetails' => true,
	//Database definition
	'db' => [
		'driver' => 'mysql',
		'host'=> 'localhost',
		'database' => 'raw-php',
		'username' => 'root',
		'password' => '',
		'charset' => 'utf8',
		'collation' => 'utf8_unicode_ci',
		'prefix' => '',
	],
	'cakeDB' => [
		'className' => 'Cake\Database\Connection',
		'driver' => 'Cake\Database\Driver\Mysql',
		'database' => 'raw-php',
		'username' => 'root',
		'password' => '',
		'cacheMetadata' => false // If set to `true` you need to install the optional "cakephp/cache" package. 'composer require cakephp/cache'

	]
  ]
]);

```



## Query the table from a controller

<figure>
{% highlight php %}
<?php

namespace App;

use Slim\Views\Twig;
use Psr\Log\LoggerInterface;
use Illuminate\Database\Query\Builder;
use Psr\Http\Message\ServerRequestInterface as Request;
use Psr\Http\Message\ResponseInterface as Response;

class WidgetController
{
    private $view;
    private $logger;
    protected $table;

    public function __construct(
        Twig $view,
        LoggerInterface $logger,
        Builder $table
    ) {
        $this->view = $view;
        $this->logger = $logger;
        $this->table = $table;
    }

    public function __invoke(Request $request, Response $response, $args)
    {
        $widgets = $this->table->get();

        $this->view->render($response, 'app/index.twig', [
            'widgets' => $widgets
        ]);

        return $response;
    }
}
{% endhighlight %}
<figcaption>Figure 5: Sample controller querying the table.</figcaption>
</figure>

### Query the table with where

<figure>
{% highlight php %}
...
$records = $this->table->where('name', 'like', '%foo%')->get();
...
{% endhighlight %}
<figcaption>Figure 6: Query searching for names matching foo.</figcaption>
</figure>

### Query the table by id

<figure>
{% highlight php %}
...
$record = $this->table->find(1);
...
{% endhighlight %}
<figcaption>Figure 7: Selecting a row based on id.</figcaption>
</figure>

## More information

[Eloquent](https://laravel.com/docs/5.1/eloquent) Documentation
