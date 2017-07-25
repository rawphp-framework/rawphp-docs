---
title: Installation
---

## System Requirements

* Web server with URL rewriting
* PHP 5.5 or newer

## How to Install Composer

There are two ways to install RawPHP:
* The first way is to clone `https://github.com/rawphp-framework/rawphp.git` into your local machine, then CD into it and run `composer install` in your command line. If you don't have composer already installed in your system, do download and installed  [Composer](https://getcomposer.org/) . 

* The second way is to use [Composer](https://getcomposer.org/) to install RawPHP.
Navigate to the folder you wish to install RawPHP, then run the below code in your command line
```
$ composer require partner/rawphp
```

Both methods install RawPHP and all required dependencies. RawPHP requires PHP 5.5.0 or newer.


## Setup the database
Create a new database in your database manager. Enter the database details in `config/DatabaseConfig.php`. Fill the same details for both of the two database engines even if you will end up loading up one for use.
In the root folder of the installed RawPHP application, you will see a `rawphp-database.sql` file. Run it in your newly created database.
You can read up more about this step in the [Database section](https://github.com/daveozoalor/RawPHP-docs/blob/master/docs/cookbook/database-eloquent.md)


# Connect your database
Your setup is not complete until you connect your database. 
* In the root folder of your RawPHP application, you will see an sql file named `rawphp-database.sql`. Create a database in your database management system (eg. PHPMyadmin), then run that sql file to create a `users` table. 
* Specify your database details in `config/DatabaseConfig.php`. You need to fill the same database details for the two database settings you will see in that file.

## Running your RawPHP application 
Navigate into your newly installed RawPHP folder, open a command prompt from there and run `php -S localhost:8001 -t public`. 
Then go to your browser and visit `localhost:8001`. 
Alternatively if you have a server like wampserver installed in your machine, you can install RawPHP in your server's root folder. In this case it is the www/. Start the server, then visit `localhost/your-rawphp-folder/public` on your browser.

That's it, your RawPHP application should be up and running now.

Next, you need to take the blog tutorial to get a good grip of RawPHP ASAP.

## Take the blog tutorial

To get a quick grip around RawPHP, you can take the [RawPHP Blog Tutorial](https://github.com/daveozoalor/RawPHP-docs/blob/master/docs/tutorial/first-app.md)


### Run Your Application With PHP's Webserver

This is my preferred "quick start" option because it doesn't rely on anything else!  From the `src/public` directory run the command:

    php -S localhost:8080

This will make your application available at http://localhost:8080 (if you're already using port 8080 on your machine, you'll get a warning.  Just pick a different port number, PHP doesn't care what you bind it to).

**Note** you'll get an error message about "Page Not Found" at this URL - but it's an error message **from** RawPHP, so this is expected.  Try http://localhost:8080/hello/joebloggs instead :)

### Run Your Application With Apache or nginx

To get this set up on a standard LAMP stack, we'll need a couple of extra ingredients: some virtual host configuration, and one rewrite rule.

The vhost configuration should be fairly straightforward; we don't need anything special here.  Copy your existing default vhost configuration and set the `ServerName` to be how you want to refer to your project.  For example you can set:

    ServerName rawphpproject.dev

    or for nginx:

    server_name rawphpproject.dev;

Then you'll also want to set the `DocumentRoot` to point to the `public/` directory of your project, something like this (edit the existing line):

    DocumentRoot    /home/lorna/projects/rawphp/project/src/public/

    or for nginx:

    root    /home/lorna/projects/rawphp/project/src/public/


**Don't forget** to restart your server process now you've changed the configuration!

There is also a `.htaccess` file in my `/public` directory; this relies on Apache's rewrite module being enabled and simply makes all web requests go to index.php so that RawPHP can then handle all the routing for us.  Here's the `.htaccess` file:

```
RewriteEngine on
RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule . public/index.php [L]
```

nginx does not use `.htaccess` files, so you will need to add the following to your server configuration in the `location` block:

```
if (!-e $request_filename){
    rewrite ^(.*)$ /index.php break;
}
```

*NOTE:* If you want your entry point to be something other than index.php you will need your config to change as well. `api.php` is also commonly used as an entry point, so your set up should match accordingly. This example assumes your are using index.php.

With this setup, just remember to use http://rawphpproject.dev instead of http://localhost:8080 in the other examples in this tutorial.  The same health warning as above applies: you'll see an error page at http://rawphpproject.dev but crucially it's *RawPHP's* error page.  If you go to http://rawphpproject.dev/hello/joebloggs then something better should happen.

