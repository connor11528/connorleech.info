---
layout: post
title: Laravel 5.3 Form Helpers and CRUD Controller Methods
date: "2017-01-08"
categories: ["Laravel"]
lede: "Post going through how to implement Laravel Collective Forms & HTML
helpers within a Laravel 5 app"
author: Connor Leech
published: true
---

![](https://cdn-images-1.medium.com/max/800/1*QZ8Fkp0LAPd0k9GR0UOaLA.png)

In this tutorial we are going to build forms in Laravel for creating and editing
our data. We will also define all the controller methods required for CRUD
operations.

Right now we have out form field in a **home.blade.php file**. We have the
Candidate controller and model set up in PHP on the server side, now it is time
to render our markup using Blade Templates.

> If you want to follow along the source code is available here:
> [https://github.com/connor11528/laravel-5.3-app](https://github.com/connor11528/laravel-5.3-app)
(Give it 🌟 on the githubs!)

> This is the fourth article working on this codebase. We [added
> authentication](https://medium.com/@connorleech/generate-authentication-for-a-laravel-5-3-web-app-384781a5529f#.2h2lqj9cd),
made a [typeahead Google Maps
component](https://medium.com/@connorleech/build-google-maps-typeahead-functionality-with-vue-js-and-laravel-5-3-b75986c77df1#.tnmrntx6q)
and [set up a model +
controller](https://medium.com/@connorleech/create-a-database-model-and-controller-in-laravel-5-3-b3e15218f6ae#.pe0b5pnif)
in the previous articles.

### Use Laravel 5.3 Form Helpers

[Laravel Collective](https://laravelcollective.com/) maintains [Forms & HTML
helpers](https://laravelcollective.com/docs/5.3/html) that have been removed
from the core framework but are widely used. It helps us write less code and
build HTML forms. Install the tool with composer:

```
composer require "laravelcollective/html":"^5.3.0"
```

Next, add your new provider to the  array of **config/app.php**:

```
'providers' => [
    // ...
    Collective\Html\HtmlServiceProvider::class,
    // ...
  ],
```

Finally, add two class aliases to the  array of **config/app.php**:

```
'aliases' => [
    // ...
      'Form' => Collective\Html\FormFacade::class,
      'Html' => Collective\Html\HtmlFacade::class,
    // ...
  ],
```

### Create the View Files

Let’s create generic files for adding, editing and viewing candidates. You can
create these files manually or use the command line as listed below.

    $ mkdir resources/views/candidates
    $ cd resources/views/candidates
    $ touch create.blade.php edit.blade.php form.blade.php index.blade.php show.blade.php

Our views folder now looks something like this:

![](https://cdn-images-1.medium.com/max/800/1*MK0X84eyzod70HnpaEQjTg.png)

<span class="figcaption_hack">Create the files. I pimped out my sublime colors thanks to Jeffrey Way’s
[Sublime Text Laracast
episode](https://laracasts.com/series/sublime-text-mastery/episodes/1)!</span>

### Create the Candidate Controller methods

In the [last
post](https://medium.com/@connorleech/create-a-database-model-and-controller-in-laravel-5-3-b3e15218f6ae#.c6d2e9s7p)
we added a candidate using tinker and mocked out our controller methods but did
not fill them out. Below is our CandidateController endpoints as mocked out by
Resource Controller. Check out the [Eloquent
docs](https://laravel.com/docs/5.3/eloquent) and this [Scotch.io
post](https://scotch.io/tutorials/simple-laravel-crud-with-resource-controllers)
for more info about CRUD with Resource Controllers.

Overall we only have to write about 15 lines code. This is enough to Create,
Update, Show and Edit data and render our html views. Thank you Laravel.

**app/Http/Controllers/CandidateController.php**

    <?php

    namespace App\Http\Controllers;

    use Request;
    use App\Candidate;

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

            return view('candidates.index', compact('candidates'));
        }

    /**
         * Show the form for creating a new resource.
         *
         * 
     \Illuminate\Http\Response
         */
        public function create()
        {
            return view('candidates.create');
        }

    /**
         * Store a newly created resource in storage.
         *
         * 
      \Illuminate\Http\Request  $request
         * 
     \Illuminate\Http\Response
         */
        public function store(Request $request)
        {
            $input = Request::all();
            Candidate::create($input);

            return redirect('candidates');
        }

    /**
         * Display the specified resource.
         *
         * 
      int  $id
         * 
     \Illuminate\Http\Response
         */
        public function show($id)
        {
            $candidate = Candidate::findOrFail($id);

            return view('candidates.show', compact('candidate'));
        }

    /**
         * Show the form for editing the specified resource.
         *
         * 
      int  $id
         * 
     \Illuminate\Http\Response
         */
        public function edit($id)
        {
            $candidate = Candidate::findOrFail($id);

            return view('candidates.edit', compact('candidate'));
        }

    /**
         * Update the specified resource in storage.
         *
         * 
      \Illuminate\Http\Request  $request
         * 
      int  $id
         * 
     \Illuminate\Http\Response
         */
        public function update(Request $request, $id)
        {
            $candidate = Candidate::findOrFail($id);
            $input = Request::all();
            $candidate->update($input);

            return redirect('candidates');
        }

    /**
         * Remove the specified resource from storage.
         *
         * 
      int  $id
         * 
     \Illuminate\Http\Response
         */
        public function destroy($id)
        {
            $candidate = Candidate::findOrFail($id);
            $candidate->delete();
            
            return redirect('candidates.index');
        }
    }

Note that at the top of the file we use Request instead of
Illuminate\Http\Request. This is from these posts on
[Stack](http://stackoverflow.com/questions/28573860/laravel-requestall-should-not-be-called-statically)
[Overflow](http://stackoverflow.com/questions/34675057/undefined-method-in-requestall).
Redirecting to ‘candidates’ automatically renders the index file in the
candidates directory.

![](https://cdn-images-1.medium.com/max/800/1*FvbgpP0z_jaNUwp-CUKSBg.jpeg)

Thank you Stack Overflow!

### Protect from Mass Assignment Vulnerabilities on the Model

We have to tell Laravel that it is okay for users to edit the fields in our
database. To do this add a protected field to the model.

**app/Candidate.php**

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Candidate extends Model
    {
        // protect from mass assignment vulnerabilities
        protected $fillable = [ 'name', 'email', 'phone', 'latitude', 'longitude', 'street', 'city', 'state', 'zip'];
    }

If you’re interested to learn more check the [docs on mass
assignment](https://laravel.com/docs/5.3/eloquent#mass-assignment).

### Display Candidate data from the database on the webpage

Our views inherit from **resources/views/layouts/app.blade.php**. All the
content will be rendered in the section in that file where the
*@yield(‘content’) *line is.

We want to get the data from the database on to the screen. The candidates index
file will show a list of all the candidates and the show file will show the data
for a single candidate. These view will be rendered at
[http://localhost:8000/candidates](http://localhost:8000/candidates) and
[http://localhost:8000/candidates/1](http://localhost:8000/candidates/1).

**resources/views/candidates/index.blade.php**

    ('layouts.app')

    ('content')
     <h1>Candidates</h1>
     <div>
      
    ($candidates as $candidate)
       <h2>
        <a href="{{ url('/candidates', $candidate->id) }}">
         {{ $candidate->name }}
        </a>
       </h2>
       <a href="{{ route('candidates.edit', $candidate->id) }}">
        Edit candidate
       </a>
       <div class='body'>
        <pre>{{ $candidate }}</pre>
       </div>
       
      
       <p>There are no candidates to display!</p>
      
     </div>

The [@forelse loop](https://laravel.com/docs/5.3/blade#loops) is an elegant
solution in Blade for handling cases when the array of data is empty. The url
and route methods are different ways of generating links. The `<pre>` tag is for
visualizing the javascript object on the page. It is not pretty but it shows the
data for us developer types.

![](https://cdn-images-1.medium.com/max/600/1*G68yPqTC-27usXbqPj-3Rw.jpeg)

**candidates/show.blade.php**

    ('layouts.app')

    ('content')
     <h1>{{ $candidate->name }}</h1>

    <div class='body'>
      <pre>{{ $candidate }}</pre>
     </div>


### Add Candidates and Reusing Forms

You may have noticed that we created a **form.blade.php**. This is a view
partial that we will call in both the create and edit views.

**resources/views/candidates/create.blade.php**

    ('layouts.app')

    ('content')

    <div class='col-md-6 col-md-offset-3'>
      <h1>Add New Candidate</h1>

    <hr>
      
      {!! Form::open(['action' => 'CandidateController@store']) !!}
       
    ('candidates.form', ['submitButtonText' => 'Add Candidate'])
      {!! Form::close() !!}
     </div>

**resources/views/candidates/edit.blade.php**

    ('layouts.app')

    ('content')

    <div class='col-md-6 col-md-offset-3'>
      <h1>Edit Candidate</h1>

    <hr>
      
      {!! Form::model($candidate, ['method' => 'PATCH', 'action' => ['CandidateController@update',$candidate->id]]) !!}
       
    ('candidates.form', ['submitButtonText' => 'Edit Candidate'])
      {!! Form::close() !!}
     </div>

When we click the create button we’ll get a blank form that goes to the create
method in the controller. When we go to the edit page the form will be auto
populated with the current data of the record we are editing. The
$submitButtonText is defined so the button has the appropriate “create” or
“edit” terminology. The form itself is defined below.

**resources/views/candidates/form.blade.php**

    <div class='form-group'>
     {!! Form::label('name', 'Name:') !!}
     {!! Form::text('name', null, ['class' => 'form-control']) !!}
    </div>

    <div class='form-group'>
     {!! Form::label('email', 'Email:') !!}
     {!! Form::email('email', null, ['class' => 'form-control']) !!}
    </div>

    <div class='form-group'>
     {!! Form::label('phone', 'Phone:') !!}
     {!! Form::text('phone', null, ['class' => 'form-control']) !!}
    </div>

    <div class='form-group'>
     {!! Form::label('latitude', 'Latitude:') !!}
     {!! Form::text('latitude', null, ['class' => 'form-control']) !!}
    </div>

    <div class='form-group'>
     {!! Form::label('longitude', 'Longitude:') !!}
     {!! Form::text('longitude', null, ['class' => 'form-control']) !!}
    </div>

    <div class='form-group'>
     {!! Form::label('city', 'City:') !!}
     {!! Form::text('city', null, ['class' => 'form-control']) !!}
    </div>

    <div class='form-group'>
     {!! Form::label('state', 'State:') !!}
     {!! Form::text('state', null, ['class' => 'form-control']) !!}
    </div>

    <div class='form-group'>
     {!! Form::label('zip', 'Zip:') !!}
     {!! Form::text('zip', null, ['class' => 'form-control']) !!}
    </div>

    <div class='form-group'>
     {!! Form::submit($submitButtonText, ['class' => 'btn btn-lg btn-success form-control']) !!}
    </div>

Note here that we are not using our fancy Vue.js typeahead component here.
Ideally we will have the **<location-input>** component handle filling out the
latitude, longitude, state, address and zip. That component is defined in
**resources/assets/js/components/LocationInput.vue.**

![](https://cdn-images-1.medium.com/max/800/1*AcY-VKyzhRkdG0JSxEryjQ.png)

Laravel + Vue.js = Laravue

*****

I am still figuring out how these Laravel and Vue.js games work. If you have any
suggestions (especially about using the Vue.js component with Laravel Form
Builders) let me know.

#### The source code is available here:
[https://github.com/connor11528/laravel-5.3-app](https://github.com/connor11528/laravel-5.3-app)
(Give it 🌟 on the githubs!)

Thank you for reading! I am avail on twitter
[@connor11528](https://twitter.com/Connor11528). If you enjoyed post give it a
recommend. Thanks again

<hr>

Originally published [on Medium](https://medium.com/@connorleech/create-and-edit-records-form-reuse-in-laravel-5-3-f70a4b1d5f9b)