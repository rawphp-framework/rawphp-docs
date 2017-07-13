---
title: Installation
---

## System Requirements

* Web server with URL rewriting
* PHP 5.5 or newer

## How to Install Composer
Don't have Composer? It's easy to install by following the instructions on their [download](https://getcomposer.org/download/) page.

## How to Install RawPHP

We recommend you install Slim with [Composer](https://getcomposer.org/).
Navigate into your project's root directory and execute the bash command
shown below. This command downloads the RawPHP Framework and its third-party
dependencies into your project's `vendor/` directory.

```
composer require partner/rawPHP "^1.0"
```
## Setup the database
Create a new database in your database manager. Enter the database details in `config/DatabaseConfig.php`. Fill the same details for both of the two database engines even if you will end up loading up one for use.
In the root folder of the installed RawPHP application, you will see a `rawphp-database.sql` file. Run it in your newly created database.
You can read up more about this step in the [Database section](https://github.com/daveozoalor/RawPHP-docs/blob/master/docs/cookbook/database-eloquent.md)

## Running your RawPHP 
Navigate into your newly installed RawPHP folder, open a command prompt from there and run `php -S localhost:8000` or  `php -S localhost:8000 -t public`. 
Then go to your browser and visit `localhost:8000`.

