---
layout: post
title: Build a todo app with Angular 1.x and AngularFire
date: "2015-10-23"
category: Javascript
lede: "This app uses Angular 1.x to build a todo application that persists data using Firebase and the AngularFire client side Javascript library. When I gave presentations at tech hubs in Tanzania, Kenya, Zambia and Zimbabwe in 2015 it covered some of this Angular and Firebase code. Great experience and thanks so much to the Firebase team for all of their support!"
author: Connor Leech
published: true
---

Hey everyone! [Firebase](https://www.firebase.com/) and angular.js are awesome tools that you should check out. Today we're going to build and deploy a persistent client side app with only front end code. I modeled this heavily off the Firebase + Angular [TodoMVC](https://github.com/tastejs/todomvc/tree/master/examples/firebase-angular). It's a great project but the code seemed a bit heavy. We're gunna use the most recent version of [angularFire](https://www.firebase.com/docs/web/libraries/angular/)  and bootstrap for styles

[SOURCE CODE](https://github.com/connor11528/angularFire-todo) is here

[DEMO](https://firebasingtodos.firebaseapp.com/) is here

<!-- more -->

### Get started
[Sign up](https://www.firebase.com/signup/) for a free firebase account. This is basically going to be our database. Somewhere deep down Firebase uses MongoDB. Our database is going to look like a neat JSON object

Create a firebase instance (called a Forge) for this project:

{<1>}![](/content/images/2014/12/Screen-Shot-2014-12-27-at-2-02-59-PM.png)

Once that's set up you can easily add [authentication](https://www.firebase.com/docs/web/libraries/angular/api.html#angularfire-user-authentication-and-management) and [security rules](https://www.firebase.com/docs/security/guide/) for real apps. We're gunna spin up some three way data binding between the $scope, view and our firebase forge, smile and call it a day

### Start coding

Setup a project and a static website:
```
$ mkdir firebase-todos && cd firebase-todos
$ touch index.html app.js style.css
```

**index.html**
```
<!doctype html>
<html lang="en" ng-app="todomvc" data-framework="firebase">
	<head>
		<meta charset="utf-8">
		<title>Firebase &amp; AngularJS todo app</title>
		<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.1/css/bootstrap.min.css">
		<style rel='stylesheet' href='style.css'>
		<style>[ng-cloak] { display: none; }</style>
	</head>
	<body>
	<h1>Firebase Todo</h1>

	<!-- AngularJS -->
	<script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.3.2/angular.min.js"></script>
	<!-- Firebase -->
	<script src="https://cdn.firebase.com/js/client/2.0.4/firebase.js"></script>
	<!-- AngularFire -->
	<script src="https://cdn.firebase.com/libs/angularfire/0.9.0/angularfire.min.js"></script>

	<script src='js/app.js'></script>
</body>
</html>
```
So here we included cdn to bootstrap for easy to use css classes and three javascript libraries. We have angular.js, firebase's javascript library and angularFire. Angularfire defines a $firebase service (link to [API docs](https://www.firebase.com/docs/web/libraries/angular/api.html)) that will make our three way data binding super simple and angulartastic. That means changes to data in the database, browser or $scope all talk to each other and react instantly without having to worry about $scope.$apply() or anything nutty. Inject the firebase service into your main angular module (**app.js** file):
```
var todomvc = angular.module('todomvc', ['firebase']);
```
This gives us access to `$firebase` service that we can inject as a dependency anywhere in our app. If you don't know what that means sit tight, it's easy.

### Add lipstick
I don't really understand how to debug or explain CSS. Copy and paste this into **style.css**:

```
.editing {
	border-bottom: none;
	padding: 0;
}
.editing .edit {
	display: block;
	width: 506px;
	padding: 13px 17px 12px 17px;
	margin: 0 0 0 43px;
}
.editing .view {
	display: none;
}
.editing:last-child {
	margin-bottom: -1px;
}
```
I extracted these relevant bits from [TodoMVC's base.css](https://github.com/tastejs/todomvc/blob/master/examples/firebase-angular/bower_components/todomvc-common/base.css). For the most part we'll use bootstrap.

### Add and delete todos
So here's an html setup of our todolist that will let us add and delte todos:

```
<div class='container' ng-controller="TodoCtrl">
	<section class='row'>
		<header>
			<h1 class='text-center'>What do you have to do?</h1>
			<form ng-submit="addTodo()">
				<input class='form-control' placeholder="What needs to be done?" ng-model="newTodo" autofocus>
			</form>
		</header>
		<section>
			<h3>All things todo</h3>
			<p>Double-click to edit a todo</p>
			<div class="list-group">
				<div ng-repeat="todo in todos" class="list-group-item">
					<div class="view">
						<label>{{todo.title}}</label>
						<button class='btn btn-danger pull-right' ng-click="removeTodo(todo)">X</button>
					</div>
				</div>
			</div>
		</section>
	</section>
</div>
```
So we need a `TodoCtrl` and to define `addTodo` and `removeTodo` methods on that controller's scope.

**app.js**
```

todomvc.controller('TodoCtrl', function TodoCtrl($scope, $firebase) {
	var fireRef = new Firebase('https://<NAME_OF_YOUR_FIREBASE>.firebaseio.com/');
	$scope.todos = $firebase(fireRef).$asArray();
	$scope.newTodo = '';

	$scope.addTodo = function(){
		var newTodo = $scope.newTodo.trim();
		if (!newTodo.length) {
			return;
		}
		// push to firebase
		$scope.todos.$add({
			title: newTodo,
			completed: false
		});
		$scope.newTodo = '';
	};
    
	$scope.removeTodo = function(todo){
		$scope.todos.$remove(todo);
	};

});
```

Most everything is going to be in this controller. fireRef creates a reference to our firebase database instance. The url I use is `https://firebasingtodos.firebaseio.com/`. It'll be whatever-you-named-your-forge.firebaseio.com/ in your code. This is kind of like an API endpoint for transorming your data. Next we save the whole database as an array an attach that to the controller's scope. In the addTodo and removeTodo bits we use methods of the firebase array `.$add()` and `.$remove()`. That's it! Now you can post and remove todos, refresh the page and everything stays the same. Open up your Forge in a different browser and you'll see your database and the browser update at the same time. Neat huh? This only scratches the surface. Make sure to check out the [AngularFire API docs](https://www.firebase.com/docs/web/libraries/angular/api.html) to see all the neat angularesque things you can do to your data.

### Conclusion

Most of the rest of the code for marking todos as completed and whatnot I stole from the todoMVC example. There is [one really ugly bit of code](https://github.com/jasonshark/angularFire-todo/blob/master/index.html) that I have because I couldn't get the line-through effect to work with ng-class and CSS.

```
<label ng-dblclick="editTodo(todo)" ng-if='todo.completed' ng-style="{'text-decoration': 'line-through'}">{{todo.title}}</label>
<label ng-dblclick="editTodo(todo)" ng-if='!todo.completed' ng-style="{'text-decoration': 'none'}">{{todo.title}}</label>
```

I repeated myself, used `ng-style` and `ng-if` based on the completion status of the task. This works but if you have a fix to clean this up please submit a pull request and I'll merge that shit up.


Big props to the firebase team on building an awesome product and TodoMVC for all of their work. If you have questions about the code for this tutorial you can find me on [Twitter](https://twitter.com/connor11528) or [Github](https://github.com/connor11528).
