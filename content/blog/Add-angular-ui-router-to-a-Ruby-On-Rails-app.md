---
layout: post
title: Part 3 - Add angular-ui-router to a Ruby On Rails app
date: "2015-12-14"
categories: ["Programming"]
toc: false
lede: "In this post we add client side routing to a Ruby On Rails app using Angular 1.x"
author: Connor Leech
published: true
---

**tldr;** — [source code on github](https://github.com/cleechtech/rails-task-list) (give it a star!)

In [part 1](http://connor11528.github.io/2015/12/02/Ruby-On-Rails-%E2%80%94-Introduction-for-the-total-n00b/) and [part 2](http://cleechtech.github.io/2015/12/04/Add-Angular-js-to-Ruby-on-Rails-app/) we set up a Ruby On Rails app with some static data and added angular and bower. In this post we will set up routing and request the list of tasks using angular instead of the rails template language.


### Make Rails routes

In **config/routes.rb** we are going to set up the task index template as the route. Any request that is unmatched (as a fallback) will also serve the index template. Angular is on the index template so most routing our users see will happen through angular.

```
Rails.application.routes.draw do
  root :to => "task#index"
  get "*unmatched_route" => "task#index"
 ...
```

Then in **views/task/index.html.erb** add a space where the routing will occur:

```
<div class=’container’>
 <div ui-view></div>
</div>
```

Next up we are going to use [angular-rails-templates](https://github.com/pitr/angular-rails-templates). In the **Gemfile** add

```
gem ‘angular-rails-templates’
gem ‘sprockets’, ‘2.12.3’
```

In order for this gem to work we need at least sprockets 2.12. The default sprockets for a rails 4 app (as of this writing is 2.11). You can update sprockets by running

```
$ bundle update sprockets
```

Side note, if you get an error when running bundle install with SSL certificates like:

```
Gem::RemoteFetcher::FetchError: SSL_connect returned=1 errno=0 state=SSLv3 read server certificate B: certificate verify failed (https://rubygems.org/gems/rack-1.5.5.gem)
```

then change your gemfile to `source ‘http://rubygems.org'` instead of `'https://rubygems.org'`. One letter change gets rid of the error.

### Write angular routes 

In **app/assets/javascripts/application.js** add the javascript libraries we need.

```
//= require angular/angular
//= require angular-ui-router/release/angular-ui-router
//= require angular-rails-templates
//= require_tree ../templates
```

The last line here means that our client side templates live in **app/assets/templates**. So create that folder.

Next add a module dependency to angular and define our tasks route.

```
var app = angular.module(‘rails-task-list’, [
 ‘ui.router’,
 ‘templates’
]);

app.config([
 ‘$stateProvider’, 
 ‘$urlRouterProvider’, 
 ‘$locationProvider’, 
 function($stateProvider, $urlRouterProvider, $locationProvider){
     $stateProvider.state(‘tasks’, {
          url: ‘/’,templateUrl: ‘tasks.html’,
          controller: ‘TaskCtrl’
      });

    $urlRouterProvider.otherwise(‘/’);
    $locationProvider.html5Mode({
       enabled:true,
       requireBase: false
     });
 }]);
```

Create **app/assets/templates/tasks.html**

```
<div class=’container’>
 <div class=’row text-center’>
 <h2>Task List</h2>
 <ul class=’list-group’>
 <li class=’list-group-item’ ng-repeat=’task in tasks’>
 {{task.title}}
 </li>
 </ul>
 </div>
</div>
```

Now in our controller send the request for the tasks:

```
app.controller(‘TaskCtrl’, [
 ‘$scope’,
 ‘$http’,
 function($scope, $http){
     $scope.taks = [];
     $http.get(‘/api/v1/tasks’).then(function(res){
         console.log(res.data);
         $scope.tasks = res.data;
     });
}]);
```

Send JSON with Rails Our angular routing is set up and we are ready to handle the JSON response. Now we need to set up the “/api/v1/tasks” endpoint. In **config/routes.rb** add our api definition.

```
Rails.application.routes.draw do

scope ‘/api’ do
 scope ‘/v1’ do
   scope ‘/tasks’ do
     get ‘/’ => ‘task#all’
   end
  end
 end

 root :to => “task#index”
 get “*unmatched_route” => “task#index”
end
```

This says that when we hit “/api/v1/tasks” invoke the all method in TaskController. Remember, this is our rails task ctrl, not our angular task controller. Define the all method on the TaskController in **app/controllers/task_controller.rb** so that it looks like:

```
class TaskController < ApplicationController
 def index
 end

def all
 render json: Task.all
 end

end
```

So here the “index” method will render the html page. The “all” method will send all of the tasks to the client as json, then angular will render them. To see the raw JSON point your browser to `http://localhost:3000/api/v1/tasks`.

### Conclusion

In this tutorial we set up angular.js routing for our Ruby On Rails app. We serve client side templates and fetch data asynchronously using angular.js. You can [check out the source code](https://github.com/cleechtech/rails-task-list). Give it a [star](https://github.com/cleechtech/rails-task-list/stargazers) if you find it helpful. Also you can ask me questions [on twitter](https://twitter.com/realjasonshark).