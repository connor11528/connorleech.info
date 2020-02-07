---
layout: post
title: Facebook authentication with Ionic 1.x and Firebase
date: "2015-10-24"
category: Javascript
lede: "We use Firebase and Ionic 1.x to add a login with Facebook feature to our Angular 1.x powered mobile app."
author: Connor Leech
published: true
---

Starter code is [HERE](https://github.com/connor11528/ionic-firebase-auth-starter)


<iframe src="https://ghbtns.com/github-btn.html?user=connor11528&repo=ionic-firebase-auth-starter&type=star&count=true&size=large" frameborder="0" scrolling="0" width="160px" height="30px"></iframe>

<iframe src="https://ghbtns.com/github-btn.html?user=connor11528&repo=ionic-firebase-auth-starter&type=fork&count=true&size=large" frameborder="0" scrolling="0" width="158px" height="30px"></iframe>

<i>A mobile app template built with ionic framework.</i>

### Generate app: 
```
$ $ npm install -g cordova ionic
$ ionic start ionic-firebase-auth-starter sidemenu
$ ionic platform add ios
$ ionic platform add android
$ ionic build ios
$ ionic build android
$ ionic emulate ios
$ ionic emulate android
```

For android you also must install the sdk: 
```
$ brew install android-sdk
$ android
$ android list targets
$ android avd
```

Then select the package and install it after agreeing to the license. I got a `Error: Please install Android target: "android-22"`. That's the error installing the packages should get rid of.


### testing the facebook login

To test our app in the browser run `$ ionic serve`

Normally if we were authenticating with facebook in the browser using angular.js and firebase we have a whole list of methods. Check out the [angularFire documentation](https://www.firebase.com/docs/web/libraries/angular/api.html). For auth in the browser we could use `$authWithOAuthPopup` or `$authWithOAuthRedirect`. Those do not work for a mobile application.


Fortunately, there is a cordova extension for this that works. Check out the [ngCordova OAuth plugin](http://ngcordova.com/docs/plugins/oauth/). The author of that plugin has a [legit tutorial](https://blog.nraboy.com/2015/03/sign-into-firebase-with-facebook-using-ionic-framework/), that's where I got the code.

That said, using the plugin we can't test authentication in the web browser, we have to use the emulator. If you try to go `$ ionic serve` and then click login with facebook you'll get an error in the console:

`ERROR: Cannot authenticate via a web browser`

So we'll have to test that our authentication works with 

```
$ ionic build ios
$ ionic emulate ios
```

The emulator is slow but deal with it. For our other features we'll test using `ionic serve` cause changes are reflected instantly.

### Start building

For all those notes we haven't written any code. When the user first fires up our app I want to go to a login screen, then after they log in redirect them to the app.

We need to install dependencies. Ionic comes with bower as a package manager. Bower is for installing client side libraries. 

```
$ bower install ngCordova firebase angularfire --save
```

This installs these in `www/lib` as specified in the `.bowerrc` file. The `--save` flag makes them persistent (added to `bower.json`).

We have the client side dependencies, now load them into `www/index.html` like a boss.

![](http://media.giphy.com/media/GPZJLuaiexcIg/giphy.gif)

Script tags look like whoa:

```
<!-- ionic/angularjs js -->
<script src="lib/ionic/js/ionic.bundle.js"></script>
<script src="lib/ngCordova/dist/ng-cordova.js"></script>
<script src='lib/firebase/firebase.js'></script>
<script src='lib/angularfire/dist/angularfire.js'></script>

<script src="js/ng-cordova-oauth.min.js"></script>
<!-- cordova script (this will be a 404 during development) -->
<script src="cordova.js"></script>

<!-- your app's js -->
<script src="js/app.js"></script>
<script src="js/controllers.js"></script>
```

### Add routes

Ionic uses [angular ui-router](https://github.com/angular-ui/ui-router/wiki)

```
// Ionic Starter App
// ..comments..

var app = angular.module('starter', [
  'ionic', 
  'ngCordova',
  'firebase'
]);

// ...
// more stuff
// ...

.config(function($stateProvider, $urlRouterProvider) {
  $stateProvider

  .state('app', {
    url: "/app",
    abstract: true,
    templateUrl: "templates/menu.html",
    controller: 'AppCtrl'
  })

  .state('login', {
    url: '/login',
    templateUrl: 'templates/login.html',
    controller: 'AuthCtrl'
  })

  .state('app.search', {
    url: "/search",
    views: {
      'menuContent': {
        templateUrl: "templates/search.html"
      }
    }
  })

  .state('app.browse', {
    url: "/browse",
    views: {
      'menuContent': {
        templateUrl: "templates/browse.html"
      }
    }
  })

  // if none of the above states are matched, use this as the fallback
  $urlRouterProvider.otherwise('/login');
});
```

Cool. So in the app variable declaration we add ngCordova and firebase. Those modules come from the libraries we installed via bower. Then set up a login state to render a template when the user hits `/login` route. For the mobile app obviously there is no url bar. When we test using `$ ionic serve` you will see the urls, but not in device emulation mode. The default if no route matches is to go to the login template (that's the last line of code).

Create the login template in **www/templates/login.html**. A whole bunch of stuff got generated using "sidemenu" in the beginning. It's fine to delete code.

```
<ion-view view-title="Sign-In">
  <ion-content>
    <div class="padding" style='margin-top:80px;'>
      <button class="button button-block button-positive" ng-click="login()">
        Sign in with facebook
      </button>
    </div>
  </ion-content>
</ion-view>
```

Then there's the auth controller, this is where all the work really gets done:

```
.controller('AuthCtrl', function($scope, $rootScope, $firebaseAuth, $cordovaOauth, $state){
  var fb_app_id = '1416319645359452';
  var ref = new Firebase("http://giphy.firebaseio.com/");
  var auth = $firebaseAuth(ref);
  $rootScope.user = null;

  $scope.login = function() {
    $cordovaOauth.facebook(fb_app_id, ["email"]).then(function(result) {
        auth.$authWithOAuthToken("facebook", result.access_token).then(function(authData) {
            console.log(JSON.stringify(authData));
            
            $rootScope.user = authData;
            $rootScope.$apply();
            $state.go('app.browse');

        }, function(error) {
            console.error("ERROR: " + error);
        });
    }, function(error) {
        console.log("ERROR: " + error);
    });
  };
})
```

This little gem is largely taken from [@nraboy](https://twitter.com/nraboy)'s [blog post](https://blog.nraboy.com/2015/03/sign-into-firebase-with-facebook-using-ionic-framework/). There are definite ways to clean this up, but for now you can emulate, log in with facebook and then see the user's credentials in the browse template:

```
<ion-view view-title="Browse">
  <ion-content>
    <h1>Browse</h1>

    <pre{{user | json}}</pre>
  </ion-content>
</ion-view>
```

For the record those `ion-` html tags are angular.js directives. They make all the awesome ionic styling that we get for free.

### Facebook setup

For all this to work we have to create a facebook app. The variable `fb_app_id` is my public app id. I also had to add those credentials to my firebase forge.

![](/content/images/2015/06/Screen-Shot-2015-06-02-at-10-19-02-AM.png)

You get your credentials from [developers.facebook.com/apps](https://developers.facebook.com/apps):

![](/content/images/2015/06/Screen-Shot-2015-06-02-at-10-20-23-AM.png)

Make sure to go to `Settings > Advanced` and add `http://localhost/callback` to "Valid OAuth redirect URIs":

![](/content/images/2015/06/Screen-Shot-2015-06-02-at-10-23-03-AM.png)

Now you have authentication with facebook started for a mobile app. Build something awesome.

Here is the [ionic-firebase-auth-starter](https://github.com/connor11528/ionic-firebase-auth-starter) code.

<iframe src="https://ghbtns.com/github-btn.html?user=connor11528&repo=ionic-firebase-auth-starter&type=star&count=true&size=large" frameborder="0" scrolling="0" width="160px" height="30px"></iframe>

<iframe src="https://ghbtns.com/github-btn.html?user=connor11528&repo=ionic-firebase-auth-starter&type=fork&count=true&size=large" frameborder="0" scrolling="0" width="158px" height="30px"></iframe>


Questions? Comments? hit me up on [twitter](https://twitter.com/connor11528)

![](http://media.giphy.com/media/9uBX8yeCP2Lmg/giphy.gif)