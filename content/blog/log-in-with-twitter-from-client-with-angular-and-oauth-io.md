---
layout: post
title: Client Side Twitter Login with Angular 1.x and OAuth.io
date: "2015-10-23"
category: Javascript
lede: "Implements a Twitter social login feature using OAuth.io service provider and Angular.js 1.x"
author: Connor Leech
published: true
---

Authentication using the OAuth.io service on the client side

First off thank you to Preetish Panda for his tutorial on oauth.io. You can find that tutorial [here](http://www.sitepoint.com/building-twitter-app-using-angularjs).


===============

[SOURCE CODE](https://github.com/connor11528/oauthio-example) for this tutorial.

[DEMO](http://connorleech.info/oauthio-example/#/) of async twitter authentication



### Getting started
Create a static bare-bones angular app. There is an angular.js starter app [here](https://github.com/connor11528/angular-starter).

```
$ git clone git@github.com:connor11528/angular-starter.git
$ cd angular-starter
$ python -m SimpleHTTPServer
```

If you're using git and want to push this to your own repo add it as a remote:

```
$ git remote rm origin
$ git remote add origin <your_remote_url>
$ git push origin master
```

### Create an app with twitter
We are going to use twitter to handle our authentication. That means "Log in with twitter". In order to do that we need to register an application with the good folks at twitter.

Go here: https://apps.twitter.com/app/new

![](/content/images/2015/05/Screen-Shot-2015-05-31-at-11-51-59-AM.png)

For the record you also need to have a mobile phone associated with your twitter profile before you can create an application. Go to settings/mobile to add one. https://oauth.io/auth is our Callback URL. For the website URL I am using my custome domain hosted by [Github Pages](https://pages.github.com/). It is free with your github account and is a good tool and way to showcase your work.

![](http://media.giphy.com/media/pzmTB7cwkfx0Q/giphy.gif)

### Create auth.io account

[OAuth](http://oauth.net/) is an open protocol for authorization across applications. [OAuth.io](https://oauth.io/home) is a company that makes using the OAuth protocol mad simple. The two are different. We can make 2000 API calls per month for free using oauth.io. If we make 20,000 API calls per month using oauth.io it costs $20.

To use oauth.io to handle authentication create an account with them: https://oauth.io/signup

On the dashboard click "Integrated APIs" and add the twitter app's credentials. The first one is "Consumer Key (API Key)" and the next field is "Consumer Secret (API Secret)" which both come from the "Keys and Access Tokens" panel on your twitter app registration page.


![](/content/images/2015/05/Screen-Shot-2015-05-31-at-12-20-58-PM.png)

Save changes and you should be able to successfully "Try Auth".


### Enough setup, let's code

We're going to use the oath.js library. Download it from their [github repo](https://github.com/oauth-io/oauth-js/blob/master/dist/oauth.js) and then link to the script:

`<script src='js/lib/oauth.js'></script>`

After we initialize our main angular app in `js/app.js` add a constant with your oauth.io **Public Key**.

We're also gunna define routes using [ui-router](https://github.com/angular-ui/ui-router):

```
var app = angular.module('angular-starter', [
	'ui.router'
]);

app.constant('OAUTH_PUBLIC_KEY', 'p2edLIP-Hu5-VvhkJN1T_Ncba0M');

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

Then we'll define the template. It is a simple navbar with sign in, hello message and log out buttons. We show them conditionally based on a `$rootScope` value we'll get to in a second. ng-if hides or shows based on if `$scope.currentUser` is true or false.

```
<div class='row'>
	<nav class="navbar navbar-default" role="navigation">
	   <div class="navbar-header">
	      <a class="navbar-brand" href="#">oauth.io-example</a>
	   </div>
	   <div>
	      <ul class="nav navbar-nav pull-right">
	        <li>
	        	<a href ng-if='!currentUser' ng-click="signIn()">Sign in with twitter</a>
	        </li>
	    	<li>
	    		<a href ng-if='currentUser'>Hello {{currentUser.name}}</a>
	    	</li>
	    	<li>
	    		<a href ng-if='currentUser' ng-click="signOut()" id="signOut">Sign Out</a>
	    	</li>
	      </ul>
	   </div>
	</nav>
</div> 

<div class='row'>
	<pre{{ currentUser | json }}</pre>
</div>
```

Next we'll setup MainCtrl:

```
app.controller('MainCtrl', function($scope, oauth){
	$scope.user = {};

	//when the user clicks the connect twitter button, the popup authorization window opens
    $scope.signIn = function() {
        oauth.connectTwitter();
    };
 
    //sign out clears the OAuth cache, the user will have to reauthenticate when returning
    $scope.signOut = function() {
        oauth.clearCache();
    };

    //if the user is a returning user, hide the sign in button and display the tweets
    if (oauth.isReady()) {
    	// gets data and updates $rootScope
        oauth.getUserData();
    }
});
```

All of the work will really be happening in our oauth service.

### oauth service

```

app.service('oauth', function(OAUTH_PUBLIC_KEY, $q, $http, $rootScope){
	var authorizationResult = false;
 
    return {
        initialize: function() {
            $rootScope.currentUser = false;

            //initialize OAuth.io with public key of the application
            OAuth.initialize(OAUTH_PUBLIC_KEY, {
                cache: true
            });
            //try to create an authorization result when the page loads,
            // this means a returning user won't have to click the twitter button again
            authorizationResult = OAuth.create("twitter");
        },
        isReady: function() {
            return (authorizationResult);
        },
        connectTwitter: function() {
            var deferred = $q.defer();
            var self = this;

            OAuth.popup("twitter", {
                cache: true
            }, function(error, result) {
                if (!error) {
                    // update token
                    authorizationResult = result;

                    // get data from twitter
                    self.getUserData();
                    deferred.resolve();
                    
                } else {
                    // shit shit fire ze missiles
                    deferred.reject();
                    console.error('could not connect to twitter');
                    $rootScope.currentUser = false;
                }
            });

            return deferred.promise;
        },

        getUserData: function(){
            var oauth_token = this.isReady();
            if(oauth_token){
                // get user's data from token
                oauth_token.me().done(function(currentUser){
                    $rootScope.currentUser = currentUser;

                    // run the digest cycle to update scope with the 
                    // data we got from .me() oauth.io method
                    $rootScope.$apply();
                });
            } else {
                console.error('Cannot get user data. No one is logged in!');
                return false;
            }
        },
        clearCache: function() {
            // log them out and clear the scope
            OAuth.clearCache('twitter');
            $rootScope.currentUser = false;
        }
    };
});
```

![](http://media.giphy.com/media/HZCVwHeM5UIrS/giphy.gif)

In the `getUserData` function I get the logged in user's information via [.me()](http://docs.oauth.io/?Javascript#user-data-api) otherwise their info is only the token. We update the $rootScope and then have to run the digest cycle again with $apply() to let angular know about the changes. 


This is not perfect. The client could change the value of `$rootScope` and nothing would really be hidden. This does not really secure our app but it does get data from twitter for us. oauth.io has hundreds of services that we could get information from. Log in to the dashboard, click "Integrated APIs" then "Add APIs" and type any provider you can think of. If we wanted to associate our own data with a user's account we would need our own database and server, or we could use [firebase](https://www.firebase.com/)

Here is the completed [DEMO](http://connorlee.ch/oauthio-example/#/). 

Add your github url (`<username>.github.io`) to "Domains & URLs whitelist" on the oauth.io dashboard homepage and you can have a live site also.


final [SOURCE CODE](https://github.com/connor11528/oauthio-example). Give it a star!
<iframe src="https://ghbtns.com/github-btn.html?user=connor11528&repo=oauthio-example&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>

![](http://media3.giphy.com/media/14up2cTMOGbXPO/giphy.gif)
