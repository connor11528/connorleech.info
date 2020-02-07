---
layout: post
title: Deploy a Laravel 5 app to Heroku
date: "2017-08-29"
category: Cloud
description: "Get your Laravel 5 app live with a MySQL database using Heroku and PHP"
author: Connor Leech
published: true
---

![](https://bosnadev.com/wp-content/uploads/2014/09/laravel_heroku.jpg)

Heroku is an awesome platform originally built on top of AWS and currently owned by Salesforce. Laravel is a powerful PHP web framework for building applications.

In this short tutorial we're going to generate an app from [laravel-5-boilerplate](https://github.com/rappasoft/laravel-5-boilerplate) and deploy it to Heroku.

By the end of the tutorial we will have a user management and permission system built with Laravel, Bootstrap and live to the internets powered by Heroku.

If you prefer to have a generic Laravel 5 app instead of a massive boilerplate [follow this gist](https://gist.github.com/connor11528/fcfbdb63bc9633a54f40f0a66e3d3f2e). Besides step 1 everything else will be the same.

Without further ado, let's get started.

### Steps:

- Download Laravel 5 Boilerplate according to the [Quick Start Documentation](http://laravel-boilerplate.com/5.4/start.html). Also `git init` for the project.

- Create a local MySQL database for development:

```
$ mysql -uroot -p
> create database MY_DATABASE_NAME;
Ctrl-C to exit
```

- Create a **Procfile** to teach Heroku how to boot up our app:

```
$ echo web: vendor/bin/heroku-php-apache2 public/ > Procfile
```

- Create a Heroku app. You will need the [Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli) installed.

```
$ heroku create YOUR_APP_NAME
```

- Add a PHP buildpack:

```
$ heroku buildpacks:set https://github.com/heroku/heroku-buildpack-php
```

- Add MySQL to our Heroku project:

```
$ heroku addons:add cleardb
```

This creates a config variable called `CLEARDB_DATABASE_URL` which you can view using the following command. Add CLEARDB_DATABASE_URL to our .env file.

```
$ heroku config | grep CLEARDB_DATABASE_URL
```

Modify **config/database.php** so that we connect to our Heroku database.

```
<?php

$url = parse_url(getenv("CLEARDB_DATABASE_URL"));

$host = $url["host"];
$username = $url["user"];
$password = $url["pass"];
$database = substr($url["path"], 1);

return [
    ...
    'connections' => [
        'mysql' => [
            'driver' => 'mysql',
            'host' => env('DB_HOST', $host),
            'port' => env('DB_PORT', '3306'),
            'database' => env('DB_DATABASE', $database),
            'username' => env('DB_USERNAME', $username),
            'password' => env('DB_PASSWORD', $password),
            'unix_socket' => env('DB_SOCKET', ''),
            'charset' => 'utf8mb4',
            'collation' => 'utf8mb4_unicode_ci',
            'prefix' => '',
            'strict' => true,
            'engine' => null,
        ],
```


- Set an app key:

```
$ php artisan key:generate --show
$ heroku config:set APP_KEY=APP_KEY_GOES_HERE
```

- Load assets over HTTPS in production. Heroku uses `https://` to host websites. If we try to load or files over `http://` then we will get a Mixed Content, insecure stylesheet or insecure script error. Therefore, head into **App/Providers/AppServiceProvider.php** and configure to load everything over HTTPS in production:

```
use Illuminate\Support\Facades\URL;

...

// Force SSL in production
if ($this->app->environment() == 'production') {
    URL::forceScheme('https');
}
```

- Push to Heroku. 

```
$ git add -A
$ git commit -m 'heroku deployment stuff'
$ git push heroku master
$ heroku open
```

Congrats! You've deployed your Laravel 5 application to the internet using Heroku.
