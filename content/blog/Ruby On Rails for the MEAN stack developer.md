---
layout: post
title: Ruby On Rails for the MEAN stack developer
date: "2015-12-02"
category: Programming
lede: "If you don't know about this whole 'Ruby On Rails' thing, start here. We create a Task database model and render the database output to a web browser."
author: Connor Leech
published: true
---

Ruby On Rails has been around since about 2006 and it is a tried and tested way to build web applications. In this tutorial we’re going to use the popular Bootstrap 3 UI library and Ruby On Rails 4 to create a server and database. The app will display a prepopulated list of tasks. In future lessons we will go through adding and modifying tasks.

**tl;dr** [SOURCE CODE](https://github.com/connor11528/rails-task-list)

**Questions?** Hit me up [on twitter](https://twitter.com/connor11528)

### Getting started with Ruby On Rails 

First step, check that you have ROR and Postgres installed, create a new project with a postgres database:

```
$ rails --version
Rails 4.1.5
$ psql --version
psql (PostgreSQL) 9.3.5
$ rails new rails-angular database=postgresql
```

That failed with a

```
$ bundle install

Fetching gem metadata from https://rubygems.org/............
Fetching additional metadata from https://rubygems.org/..
Resolving dependencies…
Gem::RemoteFetcher::FetchError: SSL_connect returned=1 errno=0 state=SSLv3 read server certificate B: certificate verify failed (https://rubygems.org/gems/rake-10.4.2.gem)
```

Off to a slow start. Checked [SO](http://stackoverflow.com/a/19151697/2031033) and ran the below commands for a fix:

```
$ rvm get stable
$ bundle install
```

`$ bundle install` is the rails version of npm install. Instead of npm modules you use ruby gems. They’re defined in the **Gemfile**.

Then set up the database with some more rails magic:

```
$ rake db:create
$ rake db:migrate
$ rails server
```

Whenever you change your schema (database) you have to run these commands so rails can write out SQL queries for us. (That’s right we’re in structured database land.)

If we open up http://localhost:3000 we will see the default ROR home page.


![](https://cdn-images-1.medium.com/max/800/1*v-NW1U1Crzi9WLQA5ZZgPA.png)


Now make a todo model, this will be a table in the database. The model fields will be some text and a boolean if the task is completed. After generating the database model run the migration command so ROR can update all our SQL.

```
$ rails generate model Task title:string completed:boolean
invoke  active_record
create    db/migrate/20151202231447_create_tasks.rb
create    app/models/task.rb
invoke    test_unit
create      test/models/task_test.rb
create      test/fixtures/tasks.yml
$ rake db:migrate
```

Okay cool now let’s display a list of tasks on the homepage. The ROR way to do this is make a Task controller. We’re going to add an ‘index’ method when we create the controller:

```
$ rails generate controller task index

create app/controllers/task_controller.rb
route get ‘task/index’
invoke erb
create app/views/task
create app/views/task/index.html.erb
invoke test_unit
create test/controllers/task_controller_test.rb
invoke helper
create app/helpers/task_helper.rb
invoke test_unit
create test/helpers/task_helper_test.rb
invoke assets
invoke coffee
create app/assets/javascripts/task.js.coffee
invoke scss
create app/assets/stylesheets/task.css.scss
```

This generated lots of crap and we’re using coffeescript like real Ruby people — ew.

Update **config/routes.rb**:

```
Rails.application.routes.draw do
  root ‘task#index’
end
```

So then even though the task controller looks like this:

```
class TaskController < ApplicationController
 def index
 end
end
```

the page still renders:


![](https://cdn-images-1.medium.com/max/600/1*d_bY0efKGMN1AAbunYQTlg.png)




Sexy.Okay so now edit **app/views/task/index.html.erb** like the Rails machine tells us.

The index file can look something like this. 

```
<div class=’container’>
 <div class=’row text-center’>
 <h1>Tasks</h1>
 </div>
</div>
```

It will be ugly cause we have no bootstrap yet.

### Add bootstrap 

The last line of your **Gemfile**:

```
gem ‘bootstrap-sass’
```

then run

```
$ bundle install
```

In **app/assets/stylesheets/application.css** add:

```
@import "bootstrap-sprockets";
@import "bootstrap";
```

All the instructions can be found in official [bootstrap-sass](https://github.com/twbs/bootstrap-sass) repo. We also have to change the application file name to a SASS extension cause… well we’re using SASS.

```
$ mv app/assets/stylesheets/application.css app/assets/stylesheets/application.scss
```

Now the word “Taks” should be centered in the page.

### Add Tasks to the database 

We want to show some tasks initially. We can use `$ rails console` to modify the database from the command line. Another way we can do it is with **db/seeds.rb**. In that file paste in our default Tasks:

```
tasks = Task.create([
 {title: “Save Gotham”, completed: true},
{title: “Wash the Car”, completed: false},
{title: “Clean my room”, completed: false},
{title: “Do the Laundry”, completed: true},
{title: “Work on Mini-Project”, completed: true},
{title: “Walk the Dog”, completed: true}
])
```

Then populate the database with:

```
$ rake db:seed
```

Update the index file to render our task titles:

```
<h1>Tasks</h1>
<ul class=’list-group’>
  <% @tasks.each do |task| %>
    <li class=’list-group-item’><%= task.title %></li>
  <% end %>
</ul>
```

Okay now we have a list of tasks. In upcoming lessons we will go through adding tasks and toggling their completion status.

If you have questions [hit me up on twitter](https://twitter.com/connor11528) or [raise an issue](https://github.com/connor11528/rails-task-list/issues) on the source code. Thanks for reading!