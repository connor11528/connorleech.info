---
layout: post
title: Intro to Angular UI Router
date: "2015-10-23"
category: Javascript
lede: "This tutorial goes through client side routing fundamentals for an Angular 1.x application."
author: Connor Leech
published: true
---

For the uninitiated, behold angular-ui-router

[SOURCE CODE](https://github.com/connor11528/intro-to-ui-router)

If you're like me you've used [ui-router](https://github.com/angular-ui/ui-router/) before with angular.js. Your applications probably look something like this:

```
var app = angular.module('intro-ui-router', [
	'ui.router'
]);

app.config(function($stateProvider, $urlRouterProvider){

	$urlRouterProvider.otherwise("/");

	$stateProvider
	    .state('home', {
	      url: "/",
	      templateUrl: "templates/main.html",
	      controller: 'MainCtrl'
	    });
});
```

**templates/main.html** gets rendered in `<div ui-view></div>` of **index.html**. It's functional and it works.

#### Add data

To show off getting data and some more advanced features of ui-router I fired up an express server in [server.js](https://github.com/connor11528/intro-to-ui-router/blob/master/server.js) that serves **index.html** and a "posts" API.

For a really basic implementation, getting the posts:

```
app.controller('MainCtrl', function($scope, $http){
	$http.get('/api/posts').then(function(posts){
		
		$scope.posts = posts.data;
		console.log($scope.posts);
	})
});
```

Once the controller is instantialized it will fire a request to the appropriate endpoint. Once that request is returned with data the data will populate the `posts` array on the scope.

In **templates/main.html** we render the posts to the screen with a simple `ng-repeat`.

```
<h1>All posts</h1>
<div class='col-sm-12'>
	<ul>
		<li ng-repeat='post in posts'>
			<a ng-href='/#/posts/{{post._id}}'>{{post.title}}</a>
		</li>
	</ul>
</div>
```

This is a pretty simple implementation of routing with angular.js but it works. Recently I've learned some extra tricks I'd like to share.

![new tricks](http://media.giphy.com/media/uJG2A0WvErkGY/giphy.gif)

#### Refactor
The issue with this, having the call in the controller, is that there will be a small delay from when the page loads to when the page is actually populated with data from the server. ui-router has a solution - *resolve*.

From the [docs](https://github.com/angular-ui/ui-router/wiki):

> You can use resolve to provide your controller with content or data that is custom to the state. If any of these dependencies are promises, they will be resolved and converted to a value before the controller is instantiated and the `$stateChangeSuccess` event is fired.

Using the resolve property in our state:

```
.state('home', {
  url: "/",
  templateUrl: "templates/main.html",
  controller: 'MainCtrl',
  resolve: {
  	posts: function($http){
  		return $http.get('/api/posts').then(function(posts){
			return posts.data;
		});
  	}
  }
})
```

Resolve makes our controller more simple:

```
app.controller('MainCtrl', function($scope, posts){
	$scope.posts = posts;
});
```

The advantage of this approach is that `posts` is gauranteed to be available when the controller for the route is instantialized.

#### Show an individual post

So we have a lists of posts. Each post uses [ng-href](https://docs.angularjs.org/api/ng/directive/ngHref) so that we can use scope properties in the url path. Let's define that URL route:

```
.state('viewPost', {
	url: '/posts/:id',
	templateUrl: 'templates/post.html',
	controller: 'PostCtrl',
	resolve: {
		post: function($http, $stateParams){
			// get the post data for the page
			return $http.get('/api/posts/' + $stateParams.id).then(function(post){
				return post.data;
			});
		}
	}
})
```

The controller is light again:

```
app.controller('PostCtrl', function($scope, post){
	$scope.post = post;
});
```

And the view is simple:

```
<h1>{{post.title}}</h1>

<p>{{post.description}}</p>
```


That `/#/` for the ng-href is ugly. ui-router uses states, so we can use [ui-sref](https://github.com/angular-ui/ui-router/wiki/Quick-Reference#ui-sref) instead. ui-sref directs to a state instead of a url route. Our list of posts updates to:

```
<h1>All posts</h1>
<div class='col-sm-12'>
	<ul>
		<li ng-repeat='post in posts'>
			<a ui-sref="viewPost({id: post.id})">{{post.title}}</a>
		</li>
	</ul>
</div>
```

This way if we change the endpoint url for viewing a post we only change it in the routes definition, not all over our html.

To navigate to states in the javascript you can use `$state.go('state_name');`

For instance a simple back button on the post page:

`<div class='btn btn-primary' ng-click='goBack()'>< Back</div>`

and the method is defined in `PostCtrl`:

```
app.controller('PostCtrl', function($scope, post, $state){
	$scope.post = post;

	$scope.goBack = function(){
		$state.go('home');
	};
});
```

Now we can see an individual post and easily navigate back to the home state. The url is persistent, meaning you can bookmark it and refresh the page and see the same thing. We get that for free with ui-router, where it takes extra code in a framework like Backbone.

#### Nested views

On the home page I want to show a preview the description of the post when I click "preview". To do that we'll use a [nested view](https://github.com/angular-ui/ui-router/wiki/Nested-States-%26-Nested-Views) [without changing the url](http://stackoverflow.com/a/28248244/2031033).

We'll define a child state:

```
.state('home.preview', {
	controller: function($scope, $stateParams){
		$scope.post = $stateParams.post;
	},
	params: {
		post: 'defaultValue'
	},
	template: '{{post.description}}'
})
```

Then update **templates/main.html**:

```
<h1>All posts</h1>
<div class='col-sm-6'>
	<ul>
		<li ng-repeat='post in posts'>
			<a ui-sref="viewPost({id: post.id})">{{post.title}}</a>
			<small ui-sref='home.preview({post: post})'>preview</small>
		</li>
	</ul>
</div>
<div class='col-sm-6'>
	<!-- show a preview of the post here -->
	<div ui-view></div>
</div>
```

Now when you click on preview you'll see the post's description. There are some more tricks with ui-router like using [abstract states](https://github.com/angular-ui/ui-router/wiki/Nested-States-%26-Nested-Views#abstract-states) and declaring [multiple named views](https://github.com/angular-ui/ui-router/wiki/Multiple-Named-Views). This is an introduction to ui router and will serve you moving forward. Happy coding!

![happy happy joy joy](http://media2.giphy.com/media/33UbGsRWIZhkc/giphy.gif)

Also check out the [SOURCE CODE](https://github.com/connor11528/intro-to-ui-router).

Also you can follow me on [twitter](https://twitter.com/connor11528)!
