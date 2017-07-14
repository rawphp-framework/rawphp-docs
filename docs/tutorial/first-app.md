---
title: First Application Walkthrough
---

Creating a new RawPHP application simply starts with installing it first. So go through the [installation instructions](https://github.com/rawphp-framework/RawPHP-docs/blob/master/docs/start/installation.md)  of RawPHP before continuing this tutorial. 
The installation comes with a complete working user authentication system. In this tutorial, we will add a blogging feature to it. 

## Quick over view
The steps to adding a new feature/module to your RawPHP application. Files are not just named anyhow in RawPHP. RawPHP follows the standard naming convention of the most popular PHP frameworks so as to blend in perfectly.

* First create the table in your database. Lets call it `posts` table. Note that all database names must be small caps and plural.
* Create a model file for it in `app/Models/Post.php`. Note that all model names must be Capitalized and singular. 
* Create the controller file for it `app/Controller/PostsController.php`. All controller names must be Capitalized, plural and end with the suffix of `Controller.php`.
Inside your controller, you'll create several methods that will handle the several different pages to be displayed to the user. There is an example further down this page. An example of a method would be `public function displayPostsFromActiveUsers() { ...}`.
* Create the routes the user has to enter in the browser to access the page in `routes/routes.php`.
* Create the different files for 

The image below depicts the process described above, not different from other PHP frameworks. To developers using a framework for the first time, this might seem like alot. But practicing it once will demystify everything. It is actually the easiest way to build websites.

![image](https://user-images.githubusercontent.com/1010556/28173197-9460642a-67e5-11e7-915f-2957253c592c.png)


## Getting started

### Install RawPHP 
Follow the [installation instructions](https://github.com/rawphp-framework/RawPHP-docs/blob/master/docs/start/installation.md) here to get a copy of RawPHP on your local machine.

### Folder Structure 
Below is the foloder structure of RawPHP. The important subdolders have been expanded.
```
├── app
│   └── Auth
│   └── Controllers
│   └── Middlewares
│   └── Models
│   └── Validation
├── bootstrap
├── config
├── public
├── resources
│   └── views
│       └── auth
│       └── templates
├── routes
├── uploads
├── vendor
├── .htaccess
├── composer.json
├── composer.lock
├── CONTRIBUTING.md
├── readme.md
```

### Database setup
Following the RawPHP installation instructions, you must have setup your database correctly. In this section, you have to add the the posts table to the database. Below is the sql dump to run on your database management system to create your database.

```

--
-- Table structure for table `posts`
--

DROP TABLE IF EXISTS `posts`;
CREATE TABLE IF NOT EXISTS `posts` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `title` varchar(255) NOT NULL,
  `body` text NOT NULL,
  `user_id` int(11) NULL,
  `created_at` datetime NULL,
  `modified_at` datetime NULL,
  PRIMARY KEY (`id`)
) ENGINE=MyISAM DEFAULT CHARSET=latin1;

```
Running the above mysql query, will create the posts table for you. Your database should already have a `users` table.

### Creating a Model
Now we have to create the posts model. Go to `app/Models/` and create `Post.php` file.  Paste the below inside it. 

```
<?php 
namespace App\Models;

//At least one of the below ORM's must be enabled

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
		'title',
		'body',
		'user_id',
	];
	
	     /**
	     * Indicates if the model should be timestamped.
	     *
	     * @var bool
	     */
   	 public $timestamps = false;
	
}
```

That's all we need for now to get our application running.

### Creating a controller
We have to create the `PostsController.php` file inside `app/Controllers/`. We will be adding two methods inside it. One for retrieving the list of posts, the other for adding new posts. Paste the below inside it. 

```
<?php

namespace App\Controllers;

use App\Controllers\Controller;
use App\Models\Post;
use Respect\Validation\Validator as v; 

class AuthController extends Controller{
	
	/**
	* List all posts
	* This uses Laravel's ORM , you can read up CakePHP's ORM to see how to accomplish the same thing using cake.
	*/
	public function getIndex($request , $response){
		$posts = Post::all();
		return $this->view->render($posts,'posts/index.twig');
	}
	
	
	/**
	* Display the selected posts
	* @return
	*/
	
	public function getView($request , $response){
	//get the id that was sent from the routes.php, and find the post in the database
		$post = Post::find($request->getParam('id'));
		
		//pass the details to the view.twig file located in resources/posts/
		return $this->view->render($post,'posts/view.twig');
	}
	
	/**
	* Display the add post page
	* @return
	*/
	
	public function getAdd($request , $response){
		return $this->view->render($response,'posts/add.twig');
	}
	
	
	/**
	* Create new post
	* @return
	*/
	
	public function postAdd($request , $response){
		
		//we need to validate input before submission
		$validation = $this->validator->validate($request, [
			'title' => v::notEmpty(),	
			'body' => v::notEmpty()		
		]);
		
		//redirect if validation fails
		if($validation->failed()){
			$this->flash->addMessage('error', 'Fields cannot be left empty'); //You can also use error, info, warning
			return $response->withRedirect($this->router->pathFor('posts.add')); 
		}
		
		//for now, we can just save the title and the body. Later on, we'll save the user_id too
		$post = Post::create([
			'title' => $request->getParam('first_name'),
			'body' => $request->getParam('last_name')
		]);
		
		$this->flash->addMessage('success', 'New post added successfully'); //You can also use error, info, warning
		
		return $response->withRedirect($this->router->pathFor('posts.index'));
		
	}
}
```
Read more about how to perform different kinds of queries using (Laravel's ORM](https://laravel.com/docs/5.3/eloquent)  OR (CakePHP's ORM](https://book.cakephp.org/3.0/en/orm.html)

### List the controller
You have to list every controller you create in `config/ControllerConfig.php`. So at the bottom of the file, add the below:
```
$container['PostsController'] = function($container){
	return new \App\Controllers\PostsController($container);
};
```

### Create The Routes
Now head over to `routes/routes.php` and add the following lines at the bottom of the file. 
The `setName()` is optional if you know what you are doing. But in the code below, observe that only get requests use it.

```
//Receive a get request and display the index page
$app->get('/posts/index', 'AuthController:getIndex')->setName('posts.index');

//Display the post add page
$app->get('/posts/add', 'AuthController:getAdd')->setName('posts.add');

//Receive a post request and route it to the postAdd method in PostsController.php file
$app->post('/posts/add', 'AuthController:PostAdd');


//Display the specified post when a user visits and url like yourapp.com/posts/view/123
$app->get('/posts/view/{id}', 'AuthController:getView')->setName('posts.add');
```
