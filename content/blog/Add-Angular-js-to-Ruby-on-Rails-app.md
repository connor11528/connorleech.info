---
layout: post
title: Part 2 - Add Angular.js to Ruby on Rails app
date: "2015-12-04"
categories: ["Programming"]
toc: false
lede: "This article explains how to integrate Angular 1.x into a Ruby On Rails application."
author: Connor Leech
published: true
---


![Image from SPR Consulting](http://spr.com/wp-content/uploads/2015/08/RailsAngularBanner1.png)

In my [last ROR post](https://medium.com/@jasonshark/ruby-on-rails-introduction-for-the-total-n00b-fdc1a7f6567e#.bfte8gwl7) we set up a Rails app that displays a list from the database. Here is the [SOURCE CODE](https://github.com/cleechtech/rails-task-list). (Give it a star on github!)

In this post we will add angular to the task list.

## Disable turbolinks 

Turbolinks is a feature new to Rails 4. It uses pushState to change the address of the current page, meaning scripts do not have to be reloaded. Angular already does some of this magic, though it is [possible](http://stackoverflow.com/questions/14797935/using-angularjs-with-turbolinks) to use angular and turbolinks together. For this app we’re going to stick with only angular, no turbolinks.

Remove `//= require turbolinks` from **app/assets/javascripts/application.js**.

Next remove `“data-turbolinks-track” => true` from **app/views/layouts/application.html.erb**.

You can also remove `gem ‘turbolinks’` from the **Gemfile**.

## Add bower 

You could download libraries directly and drop them into **vendor/assets/javascripts/** but instead we’re going to use bower to do this for us. Create a file in the root called `.bowerrc` (run `$ touch .bowerrc` from the command line) and install client side dependencies (meaning angular and ui-router).

**.bowerrc** file looks like:

```
{
  "directory": "vendor/assets/components"
}
```

Use tabs to spaces in the .bowerrc file. Install dependencies with:

```
$ bower init && bower i angular angular-ui-router --save
```

This creates the bower.json file and installs angular and angular-ui-router.

Next we need to tell the rails asset pipeline to load in what bower added to **vendor/assets/components**. Go to **config/application.rb** and add this line:

```
config.assets.paths << Rails.root.join(‘vendor’, ‘assets’, ‘components’)
```

When rails loads in our javascripts and stylesheets it will also know to load in from the “components” folder.

Next tell the application which *specific files* we want to load from the components directory. Update **app/assets/javascripts/application.js**:

```
//= require jquery
//= require jquery_ujs
//= require angular/angular
//= require angular-ui-router/release/angular-ui-router
```

Remove the `//= require_tree .` line. If we have that it will ignore the folder structure we set up later. The order our files loads in matter so get rid of it!

Start the rails server and open up the console. Type “angular” and press enter. We’ve used bower to load client libraries into Rails.

## Create an Angular application

Okay so we’ve configured rails to load in the libraries, now we need to make our angular app and have that load in properly.

First attach a module to **app/views/layouts/application.html.erb**

```
<!DOCTYPE html>
<html ng-app=’rails-task-list’>
...
```

Next make a folder for controllers and factories in **app/assets/javascripts**

```
$ mkdir app/assets/javascripts/controllers app/assets/javascripts/factories
```

Tell the asset pipeline about these new directories, add to **config/application.rb**:

```
config.assets.paths << Rails.root.join(‘app’, ‘javascripts’, ‘controllers’)
 config.assets.paths << Rails.root.join(‘app’, ‘javascripts’, ‘factories’)
```

Then in **app/assets/javascripts/application.js** instantiate the angular module.

```
//= require jquery
//= require jquery_ujs
//= require angular/angular
//= require angular-ui-router/release/angular-ui-router
//= require_self
//= require controllers/task

var app = angular.module(‘rails-task-list’, [
 ‘ui.router’
]);
```

Create a TaskCtrl in **app/assets/javascripts/controllers/task.js**:

```
app.controller(‘TaskCtrl’, [
 ‘$scope’,
 function($scope){
     $scope.message = ‘I come from the angular controller!’;
 }]);
```

And finally update the task view in **app/views/task/index.html.erb**:

```
<div class=’container’ ng-controller=’TaskCtrl’>
 <div class=’row text-center’>
 <h1>Tasks</h1>

 <pre>{{message}}</pre>

 <ul class=’list-group’>
 <% @tasks.each do |task| %>
 <li class=’list-group-item’><%= task.title %></li>
 <% end %>
 </ul>
 </div>
</div>
```

If you are not following along, make sure you created the Rails task controller like in the [last post](https://medium.com/@jasonshark/ruby-on-rails-introduction-for-the-total-n00b-fdc1a7f6567e#.bfte8gwl7). In upcoming posts we will use ui-router with Ruby On Rails.







Source code: https://github.com/cleechtech/rails-task-list

Ask me questions or give feedback via comments or [on twitter](https://twitter.com/realjasonshark)!