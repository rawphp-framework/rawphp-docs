# RawPHP documentation
This is the documentation for (RawPHP Framework](https://github.com/daveozoalor/RawPHP-framework), a very powerful and robust framework that let's you write laravel, cakephp, slim and php code inside it. It still lighter and faster than many of the base frameworks it was built with. It let's individuals with different PHP backgrounds work effectively and efficiently on the same project, same controllers and same views!
 
RawPHP is easy to install.
You just need a web server and a copy of RawPHP, that’s it! 
RawPHP runs on different web servers such as Microsoft IIS, nginx, LightHTTPD.

RawPHP should work out of the box, but if it doesn't, confirm that your setup meets the below requirements

# Requirements
* HTTP Server. For example: Apache. Having mod_rewrite is preferred, but not required.
* PHP 5.6.0 or greater (including PHP 7.1).
* mbstring PHP extension
* intl PHP extension
* Tokenizer PHP Extension
* XML PHP Extension
* OpenSSL PHP Extension
* PDO PHP Extension

The mbstring extension is working by default In both XAMPP and WAMP.
In XAMPP, intl extension is included but you have to uncomment extension=php_intl.dll in php.ini and restart the server through the XAMPP Control Panel.
In WAMP, the intl extension is “activated” by default but not working. To make it work you have to go to php folder (by default) C:\wamp\bin\php\php{version}, copy all the files that looks like icu*.dll and paste them into the apache bin directory C:\wamp\bin\apache\apache{version}\bin. Then restart all services and it should be OK.
While a database engine isn’t required, we imagine that most applications will utilize one. RawPHP supports a variety of database storage engines:

MySQL (5.1.10 or greater)
PostgreSQL
Microsoft SQL Server (2008 or higher)
SQLite 3
All built-in drivers require PDO. You should make sure you have the correct PDO extensions installed.
Installing RawPHP
Before starting you should make sure that your PHP version is up to date:

`php -v`
You should have PHP 5.6.0 (CLI) or higher. Your webserver’s PHP version must also be of 5.6.0 or higher, and should be the same version your command line interface (CLI) uses.

# Installing Composer
RawPHP uses Composer,utilizes Composer to manage its dependencies. So, before using RawPHP, make sure you have Composer installed on your machine.
You can install composer by following the instructions on [their website](https://getcomposer.org/) 

# Create a RawPHP Project
Now that you’ve downloaded and installed Composer, create an empty folder for you RawPHP application anywhere in your computer.
If you have WAMP, XAMP or LAMP it is best to create it in your root folder. For instance, in WAMP, it would be `C:\wamp64\www`.

Then from inside the folder, install RawPHP using composer using the below command:

`composer create-project partner/rawphp`

if composer is not installed globally, you can use this instead:
`php composer.phar create-project partner/rawphp`

Once Composer finishes downloading the application skeleton and the core RawPHP library, you should have a functioning RawPHP application installed via Composer. 
Be sure to keep the composer.json and composer.lock files with the rest of your source code.

You can now visit the path to where you installed your RawPHP application and see the default home page. 
To change the content of the default page you are seeing, edit the file in src/resources/views/home.twig 

# Installing without composer
Although composer is the recommended installation method, there are pre-installed downloads [available on Github](https://github.com/daveozoalor/RawPHP-Framework). 
Those downloads contain the app skeleton with all vendor packages installed. 
Also it includes the composer.phar so you have everything you need for further use.

Keeping Up To Date with the Latest RawPHP Changes
By default this is what your application composer.json looks like:

`"require": {
    "partner/rawphp": "1.0.*"
}`
Each time you run php composer.phar update you will receive bugfix releases for this minor version. You can instead change this to ~3.4 to also receive the latest stable releases of the 3.x branch.
