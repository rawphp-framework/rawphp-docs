# Scafolding with RawPHP console
This is a way to create model, controller and view files with a single command from your console

## Introduction
To use the RawPHP console, make sure you have the latest version of RawPHP with console features. To confirm, check if your root folder has a sub
folder named `/bin` . `/bin` should be on the same folder level with `/app` , `/config` etc.
If `/bin` exits, you can then proceed.

If `/bin` folder doesn't exist in your root folder, you can clone this repository to your computer. Then copy the `/bin` folder from the cloned repository into your current
project. Then update the `composer.json` file in your root folder with the settings below.
Inside the `composer.json`, find and change:

```
   "autoload":{
    	"psr-4": {
    		"App\\": "app"
    		}
    },
```
to look like this :

```
   "autoload":{
    	"psr-4": {
    		"App\\": "app",
    		"Make\\": "bin/src/"
    		}
    },
  ```
  
  Secondly, add `"symfony/console": "^3.3"` to the bottom of the `require:` list in the same `composer.json` file. 
  Your `require:` list now looks like this:
  ```
   "require": {
        "php": ">=5.5.0",
        "pimple/pimple": "^3.0",
        "psr/http-message": "^1.0",
        "nikic/fast-route": "^1.0",
        "container-interop/container-interop": "^1.2",
        "psr/container": "^1.0",
        "slim/slim": "^3.8",
        "slim/twig-view": "^2.2",
        "illuminate/database": "^5.4",
        "respect/validation": "^1.1",
        "slim/csrf": "^0.8.1",
        "slim/flash": "^0.2.0",
        "cakephp/orm": "^3.4",
        "symfony/console": "^3.3"
    },
 
 ```
Finally, run these two commands in your command prompt:
* `composer install`
* `composer dump-autoload`

That's it, the below should then work for you.

## Using the console
To use the console, you have to open a command prompt from inside the /bin folder. The first command you should run is:
`php raw` . If it throws errors, then your RawPHP is not setup correctly. If your RawPHP console was setup correctly, it will display something like the below:

![image](https://user-images.githubusercontent.com/1010556/28904021-02f3a49a-7800-11e7-946f-cacac530b97f.png)

RawPHP console commands create sample models, views and controller files that you can then modify to suit your purpose, hence saving you lots of time

## Naming conventions 
RawPHP follows standard naming conventions, that is, there are certain names you must name your files in other to leverage RawPHP's power:

### Database naming convention
Your database tables must all be in smallcaps andin plural, multiple words should be joined with an underscore. The below are valid names:
 * `customer_accounts`
 * `posts`
 * `books`
 
### Model naming convention
Your models must be singular and Capitalized, multiple words are camel cased. Below are examples of corresponding models for the above tables:
 * `CustomerAccount`
 * `Post`
 * `Book`
 
### Controller naming convention
Your controller names must be capitalized, plural and must end with the `Controller` suffix. Multiple words are camel cased. Below are examples of corresponding controllers for the above models
 * `CustomerAccountsController`
 * `PostsController`
 * `BooksController`
 
### Views naming convention
Your view folder should bear the same name as your database table and must be named in the same way - small caps and plural.
The view files should bear the same name as their corresponding controller methods. Below are examples
* `/customer_accounts` folder may contain the following files
* ` add.twig`
* `create.twig`
* `checkIfCustomerAccountIsActive.twig`


## Creating Models
The code to create a model looks like the below:
```
php raw make:model <name of model>
``` 

Here is an example of a command to create a model for a table named `posts` table.

```
php raw make:model Post
```
A model file will be created in `app/Models/Post.php` . You then have to open the file in your editor and adjust it to suit your need.

## Creating Controllers
The code to create a model looks like the below:
```
php raw make:model <name of controller>
``` 

Here is an example of a command to create a model for a table named `posts` table.

```
php raw make:model PostsController
```
A model file will be created in `app/Controllers/PostsController.php` . You then have to open the file in your editor and adjust it to suit your need.


## Creating Views
The code to create a model looks like the below:
```
php raw make:model <name of view folder>
``` 

Here is an example of a command to create a model for a table named `posts` table.

```
php raw make:model posts
```
A model file will be created in `resources/views/posts/` . Four view files will be created in there `add.twig, edit.twig, view.twig and index.twig`
You then have to open the files in your editor and adjust it to suit your need.
