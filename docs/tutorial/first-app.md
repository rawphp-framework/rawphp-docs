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

The image below depicts the process described above, not different from other PHP frameworks. It's a powerful way to setup your application to make the rest of the work super easy. To developers using a framework for the first time, this might seem like alot. But practicing it once will demystify everything. It is actually the easiest way to build websites.

![image](https://user-images.githubusercontent.com/1010556/28173197-9460642a-67e5-11e7-915f-2957253c592c.png)


## Getting started

### Install RawPHP 
Be sure you have installed RawPHP in your computer. If not, follow the [installation instructions](../start/installation.md) here to get a copy of RawPHP on your local machine.

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
│   └── css
│   └── js
│   └── fonts
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
Following the RawPHP [installation instructions](../start/installation.md), you must have setup your database correctly and `users` table must have been setup too. In this section, you have to add the the posts table to the database. Below is the sql dump to run on your database management system to create your database.

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
  `updated_at` datetime NULL,
  PRIMARY KEY (`id`)
) ENGINE=MyISAM DEFAULT CHARSET=latin1;

```
Running the above mysql query, will create the posts table for you. Your database should already have a `users` table.

### Creating a Model
Now we have to create the posts model. Go to `app/Models/` and create `Post.php` file.  Paste the below inside it. 

```<?php 


namespace App\Models;

 
/**
* Extend eloquent model class 
* read more https://laravel.com/docs/5.1/eloquent
*/
use Illuminate\Database\Eloquent\Model;
/**
* To enable CakePHP's ORM, uncomment the line below
* Read more => https://book.cakephp.org/3.0/en/orm.html 
*/

//use Cake\ORM\TableRegistry; 


class Post extends Model
{
	
	//optional: define table name if different from 'posts'
	protected $table = 'posts';
	
	protected $fillable = [
		'id',
		'title',
		'body',
		'user_id',
	];

	/**
	* Every post belongs to a user. 
	* That is, every post has a user_id column which refers to the id of the user
	* So let's define the relationship below
	*/
	public function user(){
		return $this->belongsTo('App\Models\User');
	}	
}
```

That's all we need for now to get our application running.

### Creating a controller
We have to create the `PostsController.php` file inside `app/Controllers/`. We will be adding some methods inside it. 
* `add()` // create a new post
* `edit()` // update a post
* `delete()` // delete a post
* `index()` // list all posts


```
<?php

namespace App\Controllers;
use App\Controllers\Controller;
use App\Models\User;
use Respect\Validation\Validator as v; 
use App\Models\Post;

class PostsController extends Controller{
	
	/**
	* List all users
	* 
	* @return
	*/
	public function index($request, $response,  $args){

        //find all posts
        if(isset($args['user_id'])){
            $posts = Post::where('user_id',$args['user_id'] )->get();
             //get the user's details
	          $user = User::find($args['user_id']);

              return $this->view->render($response,'posts/index.twig', ['posts'=>$posts, 'user'=>$user]);
        }else{
            $posts = Post::all();
            return $this->view->render($response,'posts/index.twig', ['posts'=>$posts]);
        }

	}



	/**
	* Display a post
	* 
	* @return
	*/
	public function view($request, $response, $args){
	
	    $post = Post::find( $args['id']);
		
		return $this->view->render($response,'posts/view.twig', ['post'=>$post]);
		
	}


	
	/**
	* Create A New Post
	* 
	* @return
	*/
	public function add($request, $response,  $args){
	
        if($request->isPost()){
           
            /**
            * validate input before submission
            * @var 
            * 
            */ 
            $validation = $this->validator->validate($request, [
                'title' => v::notEmpty(),	
                'body' => v::notEmpty(),	
            ]);


		//redirect if validation fails
		if($validation->failed()){
			$this->flash->addMessage('error', 'Validation Failed!'); 
		
			return $response->withRedirect($this->router->pathFor('posts/add.twig')); 
		}
		
            $post = Post::create([
                'title' => $request->getParam('title'),
                'body' => $request->getParam('body'),
                'user_id' => $this->auth->user()->id,
            ]);

                $this->flash->addMessage('success', 'Post Added Successfully');
                //redirect to eg. posts/view/8 
                return $response->withRedirect($this->router->pathFor('posts.view', ['id'=>$post->id]));
           
        }
		return $this->view->render($response,'posts/add.twig');
		
	}

    
	
	/**
	* Edit post
	* 
	* @return
	*/
	public function edit($request, $response,  $args){
	
              //find the post
            $post = Post::find( $args['id']);

			//only admin and the person that created the post can edit or delete it.
			if(($this->auth->user()->id != $post->user_id) OR ($this->auth->user()->role_id < 3) ){
                
			$this->flash->addMessage('error', 'You are not allowed to perform this action!'); 
		
			return $this->view->render($response,'posts/edit.twig', ['post'=>$post]);

			}

        //if form was submitted
        if($request->isPost()){
        
         $validation = $this->validator->validate($request, [
                'title' => v::notEmpty(),	
                'body' => v::notEmpty(),	
            ]);
        //redirect if validation fails
		if($validation->failed()){
			$this->flash->addMessage('error', 'Validation Failed!'); 
		
			return $this->view->render($response,'posts/edit.twig', ['post'=>$post]);
		}
		
            //save Data
            $post =  Post::where('id', $args['id'])
                            ->update([
                                'title' => $request->getParam('title'),
                                'body' => $request->getParam('body')
                                ]);
            
            if($post){
                $this->flash->addMessage('success', 'Post Updated Successfully');
                //redirect to eg. posts/view/8 
                return $response->withRedirect($this->router->pathFor('posts.view', ['id'=>$args['id']]));
            }
        }
        
	    
		return $this->view->render($response,'posts/edit.twig', ['post'=>$post]);
		
	}


	/**
	* Delete a post
	* 
	* @return
	*/
	public function delete($request, $response,  $args){
		$user = Post::find( $args['id']);
		if($user->delete()){
			$this->flash->addMessage('success', 'Post Deleted Successfully');
			return $response->withRedirect($this->router->pathFor('posts.index', ['user_id'=>$this->auth->user()->id]));
		}
	}

}
```
Read more about how to perform different kinds of queries using (Laravel's ORM](https://laravel.com/docs/5.3/eloquent)  OR (CakePHP's ORM](https://book.cakephp.org/3.0/en/orm.html)

### List the controller
Everytime you create a new controller, you have to add it in the list in `config/ControllerConfig.php`. Many people forget this step and get errors. So at the bottom of the file, add the below:
```
$container['PostsController'] = function($container){
	return new \App\Controllers\PostsController($container);
};
```

### Create The Routes
Now head over to `routes/routes.php` and add the following lines at the bottom of the file. 
The `setName()` is optional if you know what you are doing. But in the code below, observe that only get requests use it.

```
//read more about routing here => https://github.com/rawphp-framework/RawPHP-docs/blob/master/docs/objects/router.md


//post routes
//display a list of all posts, or a list of posts created by a certain user
$app->get('/posts/index[/{user_id}]', 'PostsController:index')->setName('posts.index'); //Optional user_id parameter
//create a new post
$app->map(['POST', 'GET'], '/posts/add/', 'PostsController:add')->setName('posts.add');
//update a post
$app->map(['POST', 'GET'], '/posts/edit/{id}', 'PostsController:edit')->setName('posts.edit');
//view a post
$app->get('/posts/view/{id}', 'PostsController:view')->setName('posts.view');
//delete a post
$app->get('/posts/delete/{id}', 'PostsController:delete')->setName('posts.delete');


```

### Creating the view files
Now we have done all the background work, we need to build the different front-end views that our users will see.
* Step 1: Create this folder `resources/views/posts`.

Inside the folder, create the following files:
* `index.twig`
* `view.twig`
* `add.twig`
* `edit.twig`


### Inserting the contents of the different view pages
In  `resources/views/posts/add.twig`, paste the following so that our users will see a form to create new posts

```
{% extends 'templates/app.twig' %}

	{% block content %}
<div class="row">
	<div class="col-md-10">
		<div id="postlist">
			<div class="panel">
                <div class="panel-heading">
                    <div class="text-center">
                        <div class="row">
                            <div class="col-sm-9">
                                <h3 class="pull-left">Create new post</h3>
                            </div>
                            <div class="col-sm-3">
                                <h4 class="pull-right">
                                <small> {{ post.created_at }} </small>
                                </h4>
                            </div>
                        </div>
                    </div>
                </div>
                
            <div class="panel-body">
                <div class="login-form" >
          <form action="{{ path_for('posts.add') }}" method="post" autocomplete="off"> 
            <div class="form-group {{ errors.title ? 'has-error' : '' }}">
              <input type="text" class="form-control login-field"  name="title" placeholder="Enter your title" >
              <label class="login-field-icon fui-user" for="login-first-name"></label>
              {% if errors.title %}
              	<span class="help-block">{{ errors.title | first }}</span>
              {% endif %}
            </div>
            
            <div class="form-group {{ errors.body ? 'has-error' : '' }}">
              <textarea  rows="15" class="form-control login-field"  name="body" placeholder="Enter your content" > </textarea>
              
              <label class="login-field-icon fui-user" for="login-last-name"></label>
              
               {% if errors.body %}
              	<span class="help-block">{{ errors.body | first }}</span>
              {% endif %}	
            </div>
           
            <button class="btn btn-primary btn-lg" type="submit">Submit</button>
            
             {{ csrf.field | raw }}
            </form>
          </div>
            </div>
            
        </div>
        
       </div>
    </div>
    </div>
        <div class="col-md-1"></div>
        <div class="col-md-3">
        </div>
        <div class="col-md-1">
        </div>
    </div>
{% endblock %}

```

In  `resources/views/posts/index.twig`. This page will simply display the list of posts on the database and each one will be a link to the view page.

```{% extends 'templates/app.twig' %}

	{% block content %}
<div class="row">
	<div class="col-md-10">
		<div id="postlist">
			<div class="panel">
                <div class="panel-heading">
                    <div class="text-center">
                    {% if user %}
                        <div class="row">
                            <div class="col-sm-9">
                                <h3 class="pull-left">Posts by {{ user.first_name }}  {{ user.last_name }}</h3>
                            </div>
                            
                        </div>
                        {% endif %}
                    </div>
                </div>
                
            <div class="panel-body">
                
                <div class="container">


{% for post in posts %}
<div class="row col-md-8">
  <div class="span12">
    <div class="row">
      <div class="span8">
        <h4><strong><a href="{{ path_for('posts.view', {'id': post.id }) }}">{{ post.title }}</a></strong>
        <small> <a class="btn" href="{{ path_for('users.view', {'id': post.user.id }) }}">
        {{  post.user.first_name }} {{  post.user.last_name }} </a> </small>
        </h4>
       
      </div>
    </div>
    <div class="row">
    
      <div class="span10">      
        <p>
        {{ post.body| slice(0, 150 ) ~ '...' }}
         </p>
      </div>
    </div>
   
  </div>
</div>
{% endfor %}

<hr>


</div>
<hr>
</div>


            </div>
            
            <div class="panel-footer">
            </div>
        </div>
        
       </div>
       <div class="text-center">
            <a href="{{ path_for('posts.add') }}" class="btn btn-primary">New Post</a>
        </div> 
    </div>
</div>

</div>
	{% endblock %}

```

Now we have displayed the list of posts, we have to create the `resources/views/posts/view.twig`. 

```
{% extends 'templates/app.twig' %}

	{% block content %}
<div class="row">
	<div class="col-md-10">
		<div id="postlist">
			<div class="panel">
                <div class="panel-heading">
                    <div class="text-center">
                        <div class="row">
                            <div class="col-sm-9">
                                <h3 class="pull-left">{{ post.title }}
                                    <small class="pull-left "> <a class="btn" href="{{ path_for('users.view', {'id': post.user.id }) }}">
        {{  post.user.first_name }} {{  post.user.last_name }} </a> </small>
                                </h3>
                            </div>
                            <div class="col-sm-3">
                                <h4 class="pull-right">
                                <small> {{ post.created_at }} </small>
                                </h4>
                            </div>
                        </div>
                    </div>
                </div>
                
            <div class="panel-body">
                {{ post.body }} 
            </div>
            
        </div>
        
       </div>
    </div>
    <!-- Only the owner of this post and the admin can edit and delete it -->
     {% if (post.user_id == auth.user.id) or (auth.user.role_id <= 3) %}
		<div class="text-center">
            <p><a href="{{ path_for('posts.edit', {'id': post.id } ) }}" class="btn btn-primary">Edit post</a>
                </p>
                <p> <a onclick="return confirm('Are you sure you wish to delete this Post?');"  href="{{ path_for('posts.delete', {'id': post.id } ) }}" class="btn btn-warning">Delete post</a>
        
                </p>
          </div>
             
    {% endif %}
    </div>
        <div class="col-md-1"></div>
        <div class="col-md-3">
        </div>
        <div class="col-md-1">
        </div>
    </div>
{% endblock %}

```

Finally, we have to create a view for our users to edit the posts. Paste the below code in  `resources/views/posts/edit.twig`.

```
{% extends 'templates/app.twig' %}

	{% block content %}
<div class="row">
	<div class="col-md-10">
		<div id="postlist">
			<div class="panel">
                <div class="panel-heading">
                    <div class="text-center">
                        <div class="row">
                            <div class="col-sm-9">
                                <h3 class="pull-left">{{ post.title }}</h3>
                            </div>
                            <div class="col-sm-3">
                                <h4 class="pull-right">
                                <small> {{ post.created_at }} </small>
                                </h4>
                            </div>
                        </div>
                    </div>
                </div>
                
            <div class="panel-body">
                <div class="login-form" >
          <form action="{{ path_for('posts.edit', {'id': post.id}) }}" method="post" autocomplete="off"> 
            <div class="form-group {{ errors.title ? 'has-error' : '' }}">
              <input type="text" class="form-control login-field" value="{{ post.title }}"  name="title" placeholder="Enter your title" >
              <label class="login-field-icon fui-user" for="login-first-name"></label>
              {% if errors.title %}
              	<span class="help-block">{{ errors.title | first }}</span>
              {% endif %}
            </div>
            
            <div class="form-group {{ errors.body ? 'has-error' : '' }}">
    
              <textarea  rows="15" class="form-control login-field"  name="body" placeholder="Enter your content" >{{ post.body }}" </textarea>
              
              <label class="login-field-icon fui-user" for="login-last-name"></label>
              
               {% if errors.body %}
              	<span class="help-block">{{ errors.body | first }}</span>
              {% endif %}	
            </div>
           
            <button class="btn btn-primary btn-lg" type="submit">Submit</button>
            
             {{ csrf.field | raw }}
            </form>
          </div>
            </div>
            
        </div>
        
       </div>
    </div>
		<div class="text-center">
            <a href="{{ path_for('posts.add') }}" class="btn btn-primary">New Post</a>
        </div>
    </div>
        <div class="col-md-1"></div>
        <div class="col-md-3">
        </div>
        <div class="col-md-1">
        </div>
    </div>
{% endblock %}

```


This pretty much sums it up, run your application now and access the following url `your-localhost:port/posts/index` .


Wait, One more thing! We need to add a link to posts in the dropdown menu at the of the application. 

Right above the `auth.password.change` line, add the following line:
`<li><a href="{{ path_for('posts.index', {'user_id':  auth.user.id }) }}">My Posts</a></li>` . 
It will add a link to all posts by the current logged-in user.

If you have any questions, feel free to create an issue.

