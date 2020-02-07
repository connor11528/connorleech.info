---
layout: post
title: Use Resource Controller, Artisan and Tinker to set up REST API in Laravel 5.3
date: "2017-01-02"
category: Laravel
lede: "Set up a Database Model and Resource Controller to build REST API using Laravel 5 MVC pattern #allTheLingo"
author: Connor Leech
published: true
---

![](https://cdn-images-1.medium.com/max/800/1*L0jIQxoV1eNcNSlAj-faHA.png)

![](https://cdn-images-1.medium.com/max/600/1*u0lPW7MZEFFoUS4EIX9qag.png)

In the [last
post](https://medium.com/@connorleech/build-google-maps-typeahead-functionality-with-vue-js-and-laravel-5-3-b75986c77df1#.7jcaht2sd)
we built the client side of our application with Vue.js and Google Maps API.

[Before
that](https://medium.com/@connorleech/generate-authentication-for-a-laravel-5-3-web-app-384781a5529f#.lt3wnh1tr),
we generated our application and added authentication using Laravel 5.3.

**In this post** we will handle generating a model and controller for our
application. This will set up a database table and handle RESTful routing. To
save boilerplate code we’ll utilize Laravel’s [Resource
Controller](https://laravel.com/docs/5.3/controllers#resource-controllers).
Lastly we’ll test that it works by adding data to our database and querying it
using [Eloquent](https://laravel.com/docs/5.3/eloquent) and the php artisan
tinker command.

> **The source code for this application is available here:
> **[https://github.com/connor11528/laravel-5.3-app](https://github.com/connor11528/laravel-5.3-app)

### Generate the Model

For this one our model is going to be called Candidates with a name, email,
phone, lat, long, street, city, state and zip. We head to the command line and
run:

    $ php artisan make:model Candidate
    $ php artisan make:migration create_candidates_table

Then in the **database/migrations **we will find a create_candidates_table file
with a number before it. (The number is the timestamp in order to avoid
conflicting migrations.) In that file we add to up and down methods. The up
method is for creating the SQL columns in our database and the down is for
removing the table during migration rollbacks.

    <?php

    use Illuminate\Support\Facades\Schema;
    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Database\Migrations\Migration;

    class CreateCandidatesTable extends Migration
    {
        /**
         * Run the migrations.
         *
         * 
     void
         */
        public function up()
        {
            Schema::create('candidates', function (Blueprint $table) {
                $table->increments('id');
                $table->string('name');
                $table->string('email');
                $table->string('phone', 30)->nullable();
                $table->float('latitude', 10, 6);
                $table->float('longitude', 10, 6);
                $table->string('street');
                $table->string('city', 50);
                $table->string('state', 2);
                $table->string('zip', 12);
                $table->timestamps();
            });
        }

    /**
         * Reverse the migrations.
         *
         * 
     void
         */
        public function down()
        {
            Schema::drop('candidates');
        }
    }

We can then re-create the SQL table:

    $ php artisan migrate:rollback

If you get an error “no such table candidates”, then comment out the
schema::drop line and rollback. The rollback is for dropping the table but it
cannot be dropped if the table does not exist yet. Rollback runs the down and
then the up methods. It also deletes data in our database so do not use it in
production environment! Instead if changing the database structure or adding
tables create a new migrate file and run $ *php artisan migrate*.

### Generate the Controller

We are going to generate a [Resource
Controller](https://laravel.com/docs/5.3/controllers#resource-controllers) so
that all of our CRUD options work automatically. This adds GET, POST, PUT and
DELETE requests for our model as expected in a Restful API. It saves us a lot of
work! Thanks Laravel.

    $ php artisan make:controller CandidateController --resource

![](https://cdn-images-1.medium.com/max/800/1*OWWt18PDcDrWtBJUqGZHZw.png)

To register the routes for the controller we head into **routes/web.php **(this
is a new file structure in the 5.3 release) and add:

    Route::resource('candidates', 'CandidateController');

This generates a whole lot of routes for our controllers. This is from the docs:

![](https://cdn-images-1.medium.com/max/800/1*s2f8mNxcRtOR1ON4dgkI7Q.png)

<span class="figcaption_hack">Laravel 5.3 Resource Controller documentation</span>

Substitute ‘candidates’ for ‘photos’ and that’s what we have. You can also view
all routes in the application using $ *php artisan route:list.*

### Generate a database record with Tinker

![](https://cdn-images-1.medium.com/max/800/1*oBTLG6PHVQkmTr2asq4HNA.png)

<span class="figcaption_hack">We can use tinker and Eloquent commands to speak directly with our SQL database.</span>

The artisan tinker command allows us to write SQL direct to our database. We are
going to use it in order to generate our first Candidate.

    $ php artisan tinker
    >>>>  $candidate = new App\Candidate;
          $candidate->name = 'John Doe';
          $candidate->email = '
    ';
          $candidate->latitude = 37.809799;
          $candidate->longitude = -122.295232;
          $candidate->city = 'Oakland';
          $candidate->state = 'CA';
          $candidate->zip = '94607';
    >>> $candidate
    => App\Candidate {#691
    name: "John Doe",
    email: "dummyemail@gmail.com",
    latitude: 37.809799,
    longitude: -122.295232,
    city: "Oakland",
    state: "CA",
    zip: "94607",
    }
    >>> $candidate->save();
    => true

This will add one Candidate named John Doe to our database.

Now to view this candidate in the browser add an index method to
**app/Http/Controllers/CandidateController.php**:

    class CandidateController extends Controller
    {
        /**
         * Display a listing of the resource.
         *
         * 
     \Illuminate\Http\Response
         */
        public function index()
        {
            $candidates = Candidate::all();

            return $candidates;
        }

To test our REST API you can head to [http://localhost:8000/candidates](http://localhost:8000/candidates) and view
the JSON! Laravel automatically serializes the table data into JSON form.

![](https://cdn-images-1.medium.com/max/800/1*9juMI-_1EFsUZijQeTJVzg.png)

<span class="figcaption_hack">Our candidate in the database.</span>

Thank you for reading! If you enjoyed this article give it a favorite. Thanks
again.

**Source code:** [https://github.com/connor11528/laravel-5.3-app](https://github.com/connor11528/laravel-5.3-app)<br>
**Twitter:** [https://twitter.com/Connor11528](https://twitter.com/Connor11528)

<hr>

#### Related Articles from the Good People™ at Scotch.io

* [Simple Laravel CRUD with Resource Controllers](https://scotch.io/tutorials/simple-laravel-crud-with-resource-controllers)
* [Tinker with the Data in Your Laravel Apps with PHP Artisan Tinker](https://scotch.io/tutorials/tinker-with-the-data-in-your-laravel-apps-with-php-artisan-tinker)
