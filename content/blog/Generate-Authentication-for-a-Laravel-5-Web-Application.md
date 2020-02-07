---
layout: post
title: Generate Authentication for a Laravel 5.3 Web Application
date: "2016-12-27"
category: Laravel
description: "Quickly build out your authentication scaffolding with Laravel 5"
author: Connor Leech
published: true
---

![](https://cdn-images-1.medium.com/max/800/1*vszCMUscf4RSQrPWsXv3Mg.png)
<span class="figcaption_hack">The Laravel 5 Fundamentals series on Laracasts is great for getting started with
Laravel 5.0!</span>

In this tutorial we are going to generate an authentication system to register
and log users in using Laravel 5.3. We will use a SQLite database for
persistence in our application.

#### Generating the application

You must have Laravel 5.3 installed. Laravel depends on PHP >= 5.4, MySQL,
Composer and a few PHP extensions. If you do not have Laravel on your computer,
check out the [install docs](https://laravel.com/docs/5.0). In this post we will
cover setting up a Laravel 5.3 web application. Run these commands in the
terminal to generate the application:

    $ composer create-project laravel/laravel laravel-5.3-app "5.3" --prefer-dist
    $ cd laravel-5.3-app/
    $ php artisan serve

This will launch a web server in your browser. You can view it on
**localhost:8000 **in the browser.

> If you have issues getting Laravel setup feel free to [create a github issue
> here](https://github.com/connor11528/laravel-5.3-app/issues) and we can try to
help. Stack Overflow and the Laracasts forums are other great resources.

#### Adding Authentication

We want users to be able to login to our application. Users need a name, email
and password. We are not going to cover or include social authentication in our
app. To generate Login and Register functionality run the below command:

    php artisan make:auth

This will generate a UI and backend components for authentication. Next we must
setup the database.

#### Configure your environment

We are going to use SQLite for the sake of simplicity in this tutorial. Create a
new SQLite database with a blank file:

    touch database/database.sqlite

Next up, in the **.env **file change DB_CONNECTION=sqlite and DB_DATABASE to be
the absolute path to your **database.sqlite **file. If you do not know the
absolute path you can use:

    $ php artisan tinker
    Psy Shell v0.8.0 (PHP 5.6.27 — cli) by Justin Hileman
    >>> database_path(‘database.sqlite’)
    => “/Users/connorleech/Projects/laravel-5.3-app/database/database.sqlite”

In the end my .env file looks like:

    DB_CONNECTION=sqlite
    DB_HOST=127.0.0.1
    DB_PORT=3306
    DB_DATABASE=/Users/connorleech/Projects/laravel-5.3-app/database/database.sqlite
    DB_USERNAME=homestead
    DB_PASSWORD=secret

These environment variables are all used in **config/database.php **so you can
view and make changes there. Do not commit your **.env** file to version control
systems such as git. [Database config
docs](https://laravel.com/docs/5.3/database).

Once the database is configured create and run the database migrations to get
everything setup:

    $ php artisan migrate:install
    Migration table created successfully.

    $ php artisan migrate
    Migrated: 2014_10_12_000000_create_users_table
    Migrated: 2014_10_12_100000_create_password_resets_table

This creates a Users and Passwords Resets table for our authentication system.
Those migration files can be viewed in **database/migrations/**. If you would
like to add fields to the users table and do not have any production data to
save you can edit those files directly and then run:

    $ php artisan migrate:refresh

We will stick with the defaults. Congratulations! You have set up a web app with
authentication and its own database using Laravel 5.3. Head to
[http://localhost:8000/register](http://localhost:8000/register) in order to
create an account.

![](https://cdn-images-1.medium.com/max/800/1*PPz6O-y4a-Q2Jo32AuLlOQ.gif)

In order to edit the UI head into the **resources/views/ **directory. Routes are
defined in the **routes/** directory, a new location for the 5.3 release. Matt
Stauffer has an article
[here](https://mattstauffer.co/blog/routing-changes-in-laravel-5-3) with further
information about the change.

Thank you for reading! If you enjoyed this article give it a favorite. Thanks
again.

**Source code:** [https://github.com/connor11528/laravel-5.3-app](https://github.com/connor11528/laravel-5.3-app)
<br>
**Twitter:** [https://twitter.com/Connor11528](https://twitter.com/Connor11528)

<hr>

Originally published [on Medium](https://m.dotdev.co/generate-authentication-for-a-laravel-5-3-web-app-384781a5529f)

