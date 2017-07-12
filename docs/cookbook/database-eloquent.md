---
title: Using Laravel's Eloquent or CakePHP's ORM with RawPHP
---

You can use a database ORM such as [CakePHP's ORM](https://book.cakephp.org/3.0/en/orm.html) or [Eloquent](https://laravel.com/docs/5.1/eloquent) to connect your RawPHP application to a database. 
This article will teach how to set them up, for how to perform different database queries follow the link above to readup it up on their web pages depending on which of them you choose to use.

RawPHP comes with Laravel enabled by default, to use CakePHP ORM, you just have to turn it on. You can have both of them turned on at the same time, which means you can write both Cakephp and Laravel ORM in the same controller methods. Awesome, isn't it?



# Adding Laravel Eloquent and CakePHP's ORM to your application
Steps:
* Specify the database details in `config/DatabaseConfig.php`
* Enable them in  `config/ORMConfig.php`. Laravel is enabled by default, to enable CakePHP just uncomment it.
* Import them in the Model files you wish to use eg. `User.php`

## Specify the database details in `config/DatabaseConfig.php`

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

## Enable your prefered ORM in  `config/ORMConfig.php`.
Here is an example of what `config/ORMConfig.php`looks like by default. Laravel is enabled by default, to enable CakePHP too, just uncomment it.

```
<?php 

//use the src/config/DatabaseConfig.php file in

//load laravel eloquent
$capsule = new \Illuminate\Database\Capsule\Manager;
//add new database connection 
$capsule->addConnection($container['settings']['db']);
$capsule->setAsGlobal();
$capsule->bootEloquent();


/**
* Load CakePHP ORM
* Uncomment the below to start using CakePHP ORM, there will be no conflict
* 
*/

/*
//load  cakephp ORM
$capsule = new Cake\Datasource\ConnectionManager;

$capsule->setConfig('default', $container['settings']['cakeDB']);

*/
```

## Import the enabled ORM(s) into your model 
ere is a sample model setup in `app/Models/User.php`
```
<?php 
namespace App\Models;

 
/**
* Extend eloquent model class 
* read more https://laravel.com/docs/5.1/eloquent
*/
use Illuminate\Database\Eloquent\Model;
/**
* To enable CakePHP's ORM, uncomment the line below
 Read more => https://book.cakephp.org/3.0/en/orm.html 
*/

//use Cake\ORM\TableRegistry; 

class User extends Model
{
	
	//optional: define table name if different from 'users'
	protected $table = 'users';
	
	protected $fillable = [
		'first_name',
		'last_name',
		'email',
		'password'
	];
	
	public function setPassword($password){
		$this->update([
			'password' => password_hash($password, PASSWORD_DEFAULT),
		]);
	}
	
}
```
## Query the table from a controller
With the above setup, you can then use it in your controller like so: 
* Import your Model(s) 
* Start writing ORM queries anywhere in your controller
 
Here is a sample controller that comes with RawPHP out of the box `app/Controllers/AuthController'

```
<?php

namespace App\Controllers\Auth;
use App\Controllers\Controller;
use App\Models\User; <= Import your user model, it already has laravel and cakephp ORM imported
use Respect\Validation\Validator as v; 

class AuthController extends Controller{
	
	//Sample method to receive post request and sign a user up
	//Inside it you can write both cakePHP and Laravel ORM without any conflicts
	public function postSignup($request , $response){
		
		}
	}

```

### Query the table with where

Below are some examples demonstrating the type of queries you can run inside your controller using laravel or cakephp ORM

 Query searching for names matching foo.
...
$records = $this->DB->where('name', 'like', '%foo%')->get();
...


### Query the table by id
Selecting a row based on id
...
$record = $this->table->find(1);
...


## More information

Discover more queries you can run :
* Laravel => [Eloquent](https://laravel.com/docs/5.1/eloquent) Documentation
* CakePHP => [CakePHP's ORM](https://book.cakephp.org/3.0/en/orm.html)
