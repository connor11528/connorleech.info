---
layout: post
title: Make Maps with Angular 1.x and Leaflet using Directives
date: "2015-10-23"
category: Javascript
lede: "this tutorial goes into how to use the angular-leaflet directive (built by the Angular 1.x community for integrating maps into web applications. Leaflet is a Javascript mapping library."
author: Connor Leech
published: true
---

Make some maps with angular.js

This is a quick tutorial on how to add maps to your angular.js application. Using google maps API or leaflet.js by themselves is simple. Unfortunately manipulating the DOM is not always straightforward in angular.js projects. External libraries need to be packaged into angular.js directives. Fortunately there is a legit Angular community that has done the heavy lifting for us. 

We're going to use [angular-leaflet-directive](https://github.com/tombatossals/angular-leaflet-directive).

Here's the [gist](https://gist.github.com/jasonshark/a090c329185b94a19de2) of the finished code on the github.

First include all the scripts. We need the angular library, leaflet, the directive code and our script:

```
<!DOCTYPE html>
<html>
<head>
	<meta charset="UTF-8">
	<title>My Mapping App</title>
    <link rel="stylesheet" href="http://cdn.leafletjs.com/leaflet-0.7.3/leaflet.css" />
</head>
<body>

	<script src="http://cdn.leafletjs.com/leaflet-0.7.1/leaflet.js"></script>
	<script src="http://ajax.googleapis.com/ajax/libs/angularjs/1.2.6/angular.min.js"></script>
	<script src="./angular-leaflet-directive.js"></script>
	<script src='./app.js'></script>
</body>
</html>
```

Note we also included the leaflet.css in the head. Next we need to inject the directive as a dependency to our application when we initialize our app:

```
var app = angular.module("mymapingapp", [
    "leaflet-directive"
]);
```

Now link our angular module to the app. Add `ng-app="mymapingapp"` to the <html> like a pro. Also attach `ng-controller="MainCtrl"` to the body or the section where we want the map. We'll use the angular.js controller in order to provide options to the map and other goodies. We'll do this by extending the controller's scope with some properties. The angular-leaflet directive will read these properties off the controller's scope and update automatically. Here's the controller with the options:

```
app.controller('MainCtrl', ['$scope', function($scope) {
	// make map
    angular.extend($scope, {
        san_fran: {
            lat: 37.78,
            lng: -122.42,
            zoom: 13
        },
        events: {},
        layers: {
            baselayers: {
                osm: {
                    name: 'OpenStreetMap',
                    url: 'https://{s}.tiles.mapbox.com/v3/examples.map-i875mjb7/{z}/{x}/{y}.png',
                    type: 'xyz'
                }
            }
        },
        defaults: {
            scrollWheelZoom: false
        }
    });
}]);
```

As you can see we're extending the $scope object with more properties. We'll feed these to the directive like whoa:

```
<leaflet center="san_fran" markers="markers" event-broadcast="events" defaults="defaults" id='map'></leaflet>
```

There are a ton more options. Like a jackass I'm going to encourage you to check out the [documentation](http://tombatossals.github.io/angular-leaflet-directive/#!/).

You also need to give the map element a height and width. I select it using an id. Add this CSS: `#map{ height: 450px width: 450px; }`

So we have a map. Whoop dee do! It's kind of a lot of code, but we know it's the Angular Way&#0153;.

We'll make our map a bit more interesting. When we click the map let's add a marker. That is what the [events](http://tombatossals.github.io/angular-leaflet-directive/#!/examples/events) property on the controller scope is for.

We'll listen for clicks to the map. When a click goes down we'll add a new marker to the array:

```
// add marker
$scope.markers = new Array();
$scope.$on("leafletDirectiveMap.click", function(event, args){
    var leafEvent = args.leafletEvent;
    $scope.markers.push({
        lat: leafEvent.latlng.lat,
        lng: leafEvent.latlng.lng,
        draggable: true
    });
});
```

So now everytime you click the map you'll have annoying draggable markers. Adjust the properties on the controller scope and link that to the `<leaflet>` directive attributes and you have an angulartastic map!

You can peep the finished code in this [Github Gist](https://gist.github.com/jasonshark/a090c329185b94a19de2)