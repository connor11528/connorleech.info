---
layout: post
title: Animate Angular apps with ngFx
date: "2015-10-23"
categories: ["Javascript"]
lede: "In this post we do client side animations using Angular 1.x and the ngFx library"
author: Connor Leech
published: true
toc: false
---

Recently I stumbled upon [ngFx](https://github.com/Hendrixer/ngFx) as a simple way to add animations to angular apps. The animations are based off [animate.css](http://daneden.github.io/animate.css/) except they're all in javascript and packaged into an angular module. Angular 1.4 is [changing up animations](http://angularjs.blogspot.com/search/label/announcements) so this post might not be relevant in the future, but here's a simple animation solution for the present day.

*Update:* To use this tool there are some licensing restrictions. If you're charging people money to use your app [you'll need a license](http://greensock.com/standard-license). :Sad face:

If you would like to see the [source code](https://github.com/jasonshark/angular-ngfx-demo) it is on github.

[Demo](http://connorlee.ch/angular-ngfx-demo)


#### Get started

I set up a lightweight [angular starter](https://github.com/jasonshark/angular-starter) template with ui-router and no clunky build system. Clone it and add the [ngFx module](https://github.com/Hendrixer/ngFx/blob/master/dist/ngFx.js), [TweenMax](http://cdnjs.com/libraries/gsap) library and the [ngAnimate](http://cdnjs.com/libraries/angular.js/) module to take care of dependencies. You could also use bower if that is easier for you. Add the modules to the app.

```
var app = angular.module('ngfx-demo', [
	'ui.router',
	'ngFx',
	'ngAnimate'
]);
```

So to start off we're going to click a button and show a login form. I set up an additional state with the same controller.

```
.state('login', {
	url: "/login",
	templateUrl: "templates/login.html",
	controller: 'MainCtrl'
})
```

Then made a basic login form with a show form button.

**templates/login.html**
```
 <div class='row text-center'>
	<div class='btn btn-success' ng-click='showForm = !showForm'>Show form</div>
</div>
<div id='loginBox' ng-show='showForm' class='col-sm-offset-4 col-sm-4 img-rounded fx-fade-down'>
	<form>
		<h2>Login</h2>
		<div class='form-group'>
			<label for="inputEmail" class="sr-only">Email address</label>
			<input type="email" id="inputEmail" class="form-control" placeholder="Email address" required="" autofocus="">
		</div>
		<div class='form-group'>
			<label for="inputPassword" class="sr-only">Password</label>
			<input type="password" id="inputPassword" class="form-control" placeholder="Password" required="">
		</div>
		<div class="checkbox">
		  <label>
		    <input type="checkbox" value="remember-me"> Remember me
		  </label>
		</div>
		<button class="btn btn-lg btn-primary btn-block" type="submit">Sign in</button>
	</form>
</div>
```

Then there's the controller

```
app.controller('MainCtrl', function($scope){
	$scope.showForm = false;
});
```

And a little bit of css to prettify (not to animate).

```
body {
	height: 100%;
	background-color:lightblue;
}

#loginBox {
	background-color: white;
	margin-top:50px;
	box-shadow: 10px 10px 5px #888888;
	padding-bottom:20px;
}
```

So out of all that code and markup, the only bit that added animation was the `fx-fade-down` class on the html form's parent. The module uses the fx namespace, so any class prefixed with `fx` is about animation and adds javascript functionality. From the readme it says

> All dynamic animations tie into ngAnimate hooks. So you can apply them to...

- ng-hide / ng-show
- ng-include
- ng-if
- ng-view
- ui-view (if you're using ui.router)
- ng-switch
- ng-class
- ng-repeat

This means that these directives are all "animate aware"<sup><u>[1](https://docs.angularjs.org/api/ngAnimate)</u></sup>. So when the ng-show binding changes a *fade down* animation occurs on the element. Right now we trigger the change by clicking an element. So far it's pretty cool.

![](http://cdn2.crushable.com/wp-content/uploads/2013/09/Miley-Cyrus-Shocked.gif)

<small>**Pro tip:** With bootstrap `<div class="img-rounded">` will give you rounded corners on any element.</small>

#### Animate all the things

So looking at the list above, we can add animations for every time a page comes in. In **index.html**

```
 <div ui-view class='fx-bounce-right'></div>
```

We can add more complex animations with easings, speeds, directives and event hooks. This is a basic introduction to how to use this module for a project.

The source code is [HERE](https://github.com/jasonshark/angular-ngfx-demo). If you would like to build on this please submit a pull request!