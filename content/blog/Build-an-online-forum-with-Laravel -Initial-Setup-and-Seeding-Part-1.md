---
layout: post
title: Laravel 5 Initial Database Setup and Seeding
date: "2017-04-30"
category: Laravel
description: "Get Laravel, MySQL and SequelPro setup for starting a new web application. I refer back to this post all the time when starting new projects: `mysql -uroot -p` "
author: Connor Leech
published: true
---

![](https://cdn-images-1.medium.com/max/800/1*F64gVmpO9-reN5L2pRTrGg.png)
<span class="figcaption_hack">Laracasts video course — available here:
[https://laracasts.com/series/lets-build-a-forum-with-laravel/episodes/1](https://laracasts.com/series/lets-build-a-forum-with-laravel/episodes/1)</span>

**tl;dr** — Source code:
[https://github.com/connor11528/laravel-forum](https://github.com/connor11528/laravel-forum)

Thank you Jeffrey Way and Laracasts for this awesome series! The full video
tutorials are available here:
[https://laracasts.com/series/lets-build-a-forum-with-laravel](https://laracasts.com/series/lets-build-a-forum-with-laravel)

We’re going to be using Laravel 5.4.

Create application:

    $ laravel new laravel-forum
    $ cd laravel-forum
    $ composer install

#### Application Structure and Models

1.  Thread
1.  Reply
1.  User<br>  A. Thread is created by a user<br>  B. A reply belongs to a thread,
and belongs to a user.

Next we have a command to make the Thread model with a migration and a
[resourceful
controller](https://laravel.com/docs/5.4/controllers#resource-controllers).
Laravel is bundled with a command line tool called
[Artisan](https://laravel.com/docs/5.4/artisan) that assists us in building our
application.

    $ php artisan make:model Thread -mr

Next update the migration for Thread
(**database/migrations/<timestamp>_create_threads_table.php**):

    public function up()
    {
       Schema::create(‘threads’, function (Blueprint $table) {
           $table->increments(‘id’);
           $table->integer(‘user_id’);
           $table->string(‘title’);
           $table->text(‘body’);
           $table->timestamps();
       });
    }

Next up we have to interact with MySQL. If you do not have MySQL installed and
are on Mac, I wrote a tutorial available [here](https://medium.com/@connorleech/how-to-install-mysql-on-mac-osx-5b266cfab3b6).

If you have used Laravel before MySQL will be no sweat for you!

Log into MySQL:

    $ mysql -uroot -p 

Run a SQL command to create the database we will specify in our environment
file:

    MySQL [(none)]> create database forum;
    Query OK, 1 row affected (0.02 sec)

Close out of MySQL with Ctrl + C.

Create the file to store variable specific to your computer. This is covered in
the **.gitignore** file, therefore will not be checked in to git version
control.

    $ touch .env

Copy and paste in the **.env.example** file to **.env **and edit this part:

    DB_CONNECTION=mysql
    DB_HOST=127.0.0.1
    DB_PORT=3306
    DB_DATABASE=forum
    DB_USERNAME=YOUR_USERNAME_HERE
    DB_PASSWORD=YOUR_PASSWORD_HERE

Migrate the database using artisan.

    $ php artisan migrate
    Migration table created successfully.
    Migrating: 2014_10_12_000000_create_users_table
    Migrated: 2014_10_12_000000_create_users_table
    Migrating: 2014_10_12_100000_create_password_resets_table
    Migrated: 2014_10_12_100000_create_password_resets_table
    Migrating: 2017_04_21_224745_create_threads_table
    Migrated: 2017_04_21_224745_create_threads_table

Download [Sequel Pro](https://www.sequelpro.com/) if you do not have it.

![](http://i.imgur.com/9clFC7k.png)

Connect to the database using their friendly GUI.

![](https://cdn-images-1.medium.com/max/800/1*qPadnABxd2f14bgwRseOvg.png)
<span class="figcaption_hack">Use Sequel Pro to connect to MySQL in order to easily view database records.</span>

Back in the terminal, make a model, migration and a controller for Reply. We
went to have replies in our forum. (The -mc flag generates the
[migration](https://laravel.com/docs/5.4/migrations) and the
[controller](https://laravel.com/docs/5.4/controllers) for the Reply model.)

    $ php artisan make:model Reply -mc

Update the generated migration file in
**database/migrations/<timestamp>_create_replies_table.php**:

    public function up()
    {
       Schema::create(‘replies’, function (Blueprint $table) {
           $table->increments(‘id’);
           $table->integer(‘thread_id’);
           $table->integer(‘user_id’);
           $table->text(‘body’);
           $table->timestamps();
       });
    }

Run migrations again.

    $ php artisan migrate

In Sequel Pro click the refresh button (it is in bottom left corner)

![](https://cdn-images-1.medium.com/max/800/1*Xu3ITKNx5POtIMnxKnI7Fg.png)
<span class="figcaption_hack">Refresh button.</span>

You will see a new Reply table.

The next part is by far the most complicated… seeding the database.

#### Seed the Database

Seeding the database means that we are going to generate and pre-populate data
(such as Threads, Replies and Users) so our application is not empty and we have
real records to play with.

We are going to generate mock data for Threads. If you remember, in the
create_threads migration file we defined threads as having an id, user_id,
title, body and timestamps.

We use the [Faker PHP library](https://github.com/fzaninotto/Faker) to generate
this fake data.

Head into **database/factories/ModelFactory.php **and add a factory definition
for creating Threads. The file should look like this:

    $factory->define(App\User::class, function (Faker\Generator $faker) {
        static $password;

        return [
            'name' => $faker->name,
            'email' => $faker->unique()->safeEmail,
            'password' => $password ?: $password = bcrypt('secret'),
            'remember_token' => str_random(10),
        ];
    });

    $factory->define(App\Thread::class, function(Faker\Generator $faker){
       return [
         'user_id' => function(){
            return factory('App\User')->create()->id;
         },
         'title' => $faker->sentence,
         'body' => $faker->paragraph
       ];
    });

In the above snippet is automatically loaded by Laravel. The first part creates
some Users. In the second part, we define a new factory for creating Threads.
Faker generates bodies and titles. For the user_id section it creates a new
user, persists it to the database and then associates the id for that user with
that Thread.

#### Use php artisan tinker and factories to generate the data

Back at the command line fire up the tinker command. After that we are going to
make a new factory and create 50 threads with their associated users.

    $ php artisan tinker

    Psy Shell v0.8.3 (PHP 5.6.27 — cli) by Justin Hileman

    >>> factory('App\Thread', 50)->create();

    => Illuminate\Database\Eloquent\Collection {#688

    all: [

    App\Thread {#686

    user_id: 1,

    title: "Mollitia qui quos nesciunt perferendis error quam quo.",

    body: "Laborum rem sit reprehenderit voluptatem. Dolorem magnam possimus quod. Quam omnis architecto doloremque et non reiciendis et. Sit dolorum doloribus quo iure est molestiae at.",

    updated_at: "2017-04-30 22:06:47",

    created_at: "2017-04-30 22:06:47",

    id: 1,

    },

Hit refresh on Sequel Pro and you will see our generated records:

![](https://cdn-images-1.medium.com/max/800/1*lQKP_f2W_d7NrfFvTrxQZQ.png)
<span class="figcaption_hack">The generated Threads</span>

![](https://cdn-images-1.medium.com/max/800/1*pMr86zj094t5YnarnyxELg.png)
<span class="figcaption_hack">The generated Users</span>

Now our Threads and Users MySQL tables are full of data for us to use and play
with! There are 50 random threads that are associated with fake Users.

#### Add Replies to the Threads

Create another Factory definition at the bottom of the
**database/factories/ModelFactory.php** file.

    $factory->define(App\Reply::class, function(Faker\Generator $faker){
        return [
           'thread_id' => function(){
               return factory('App\Thread')->create()->id;
           },
           'user_id' => function(){
               return factory('App\User')->create()->id;
           },
           'body' => $faker->paragraph
        ];
    });

Refresh the database using Artisan. The below command will delete all of our
data and run these commands again from scratch.

    $ php artisan migrate:refresh

    Rolling back: 2017_04_21_233654_create_replies_table

    Rolled back:  2017_04_21_233654_create_replies_table

    Rolling back: 2017_04_21_224745_create_threads_table

    Rolled back:  2017_04_21_224745_create_threads_table

    Rolling back: 2014_10_12_100000_create_password_resets_table

    Rolled back:  2014_10_12_100000_create_password_resets_table

    Rolling back: 2014_10_12_000000_create_users_table

    Rolled back:  2014_10_12_000000_create_users_table

    Migrating: 2014_10_12_000000_create_users_table

    Migrated:  2014_10_12_000000_create_users_table

    Migrating: 2014_10_12_100000_create_password_resets_table

    Migrated:  2014_10_12_100000_create_password_resets_table

    Migrating: 2017_04_21_224745_create_threads_table

    Migrated:  2017_04_21_224745_create_threads_table

    Migrating: 2017_04_21_233654_create_replies_table

    Migrated:  2017_04_21_233654_create_replies_table

Here are the commands in tinker to generate our data from the Factory we wrote:

    $ php artisan tinker

    $threads = factory('App\Thread', 50)->create();
    $threads->each(function($thread){ factory('App\Reply', 10)->create([ 'thread_id' => $thread->id ]); });

These commands will recreate our Threads as we did previously and then will
generate 10 replies to each thread and associate the replies with the thread via
the thread_id.

![](https://cdn-images-1.medium.com/max/800/1*Qadw37OLDf2PFOPdhevTBg.png)
<span class="figcaption_hack">Ten Replies for each Thread. Each Reply has a value for thread_id</span>

#### Further Reading on Database Seeding

Below are some further reading resources on seeding databases and Model
Factories in Laravel 5.4.

https://laravel.com/docs/5.4/seeding

https://laravel-news.com/learn-to-use-model-factories-in-laravel-5-1

#### Conclusion

In this post we have set up the database, viewed in in Sequel Pro and mocked out
our data using Factories.

The full video is available [on Laracasts](https://laracasts.com/series/lets-build-a-forum-with-laravel/episodes/1)

Go sign up! Thanks for reading.

**Part 2** for this blog series is [available on Medium](https://medium.com/@connorleech/test-driven-development-tdd-in-laravel-b5a2bf9ab65b)
