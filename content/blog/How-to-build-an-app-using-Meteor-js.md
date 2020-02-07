---
layout: post
title: How to build an app using Meteor.js
date: "2015-10-23"
category: Javascript
description: "This app explores getting started with the Meteor Javascript Framework, circa 2015."
author: Connor Leech
published: true
---

Oh hurrow Meteor.js

Here's a quick tutorial about how to build a full stack website (front end code, server and database) using [meteor.js](https://www.meteor.com/).

Here's the final [source code](https://github.com/connor11528/leaderboard)

###### Get started
Install meteor, create an app and run it on port 3000:
```
curl https://install.meteor.com/ | sh
meteor create leaderboard
cd leaderboard
meteor
```
You'll see the default app. Open your browser to localhost:3000 and also take a look around at the code generated.

###### syntax runthrough
`{% raw %}{{> hello}}{% endraw %}` renders the `<template>` with name="hello". Meteor is very good about finding these templates. We will store ours in a directory called `client`.

`Template.hello.helpers({ ... }` Meteor helper functions match to a specific template. Helper functions make calls to the server to get data that is rendered in the template

`Template.hello.events({})` these use javascript/jQuery selectors as the keys and executes the value functions when those events take place. Again the events are listened for only with the hello template.

###### project structure
Delete all the generated files except .meteor directory. Create three new directories: `mkdir client server lib`

These are all [special directories](http://docs.meteor.com/#/basic/filestructure) for meteor. Javascript in meteor is largely isomorphic, meaning it can be run on client and server. By segregating our code into these directories we have separation of concerns. `lib` is for js code that runs on client and the server.


###### render html
Create a file client/layout.html and start the meteor server like before. Add elements to the file
```
<head>
  <title>Leaderboard</title>
</head>
<body>
  <h1>Leaderboard</h1>

</body>
```
Everything's taken care of by meteor and the layout template is magically rendered. Include a template we have not defined in the body after the `<h1>`. Add `{% raw %}{{> players_list}}{% endraw %}`

Everything breaks and the layout disappears. Check the browser's javascript console: "Uncaught Error: No such template: players_list" Let's define it!

Create a file in client/players_list.html and make a template
```
<template name="players_list">
	List of players will go here
</template>
```
###### define a collection
We will have a collection of players in our database. Meteor uses MongoDB. What's crazy is we can make database calls in our client javascript that will be executed immediately in a local copy of the database that sits in the browser called miniMongo. Bottom line is we want our collection available on the client and the server. Make a file lib/players.js and create a collection in the database
`PlayersList = new Mongo.Collection('players');`

To render and manipulate the collection we will use helper functions. Make a file client/players_list.js

```
Template.players_list.helpers({
  'player': function(){
    // descending score order
    // if score is the same sort ascending alphabetically
    return PlayersList.find({}, {sort: {score: -1, name: 1} });
  }
});
```

Here we match a variable player to the returning result from a meteor-MongoDB [find query](http://docs.meteor.com/#/basic/Mongo-Collection-find). This result we can render on the client like whoa: {{player}}. Though this will actually return a "cursor". Right now go into the browser js console and execute the query: `PlayersList.find({}, {sort: {score: -1, name: 1} })`. This will return a cursor that will be easily rendered in a moment. `PlayersList.find().fetch()` returns a cursor of all elements in the collection and then changes that into an array. The array is empty cause we have nothing in the database.
![](/content/images/2015/04/Screen-Shot-2015-04-08-at-10-38-08-AM-1.png)

Update client/players_list.html so it is ready to render a list of players
```
<ul>
    {{#each player}}
        <li class="player">
            {{name}}: {{score}} 
            <span class="increment">Upvote</span>
            <span class="decrement">Downvote</span>
            <span class="remove">Remove</span>
        </li>
    {{/each}}
</ul>
```
###### add to collection form
Add the markup to render a template in layout.html: `{% raw %}{{> add_player}}{% endraw %}`

Then create a file client/add_player.html
```
<template name="add_player">
	<form>
        <input type="text" name="player_name">
        <input type="submit" value="Add Player">
    </form>
</template>
```
You'll now see our form rendered on our site. In order to handle form submission and add to the collection we must capture the form submit event that will be fired from the `add_player` template. Make client/add_player.js

```
Template.add_player.events({
	'submit form': function(e, tmpl){
		e.preventDefault();
		var player_name = 	$(e.target).find('[name=player_name]').val();
		$(e.target).find('[name=player_name]').val('');
		
		Meteor.call('add_player', player_name)
	}
});
```
Breaking this down, the function will execute on form submit. We'll prevent the browser's default action, grab the input field value and reset the form using jQuery then we call a "[Meteor method](http://docs.meteor.com/#/basic/methods)".

Define our add player method on the server so it is secure. Add a file `server/player_methods.js`
```
Meteor.methods({
	'add_player': function(player){
		PlayersList.insert({
			name: player,
			score: 0
		});
	}
});
```
Add players with the form and they'll pop up in the browser.

###### insecure package
Meteor comes with an "insecure" package. Let's remove that. Instead it is best practice to "publish" and "subscribe" to the data you want available. In your terminal: `meteor remove insecure`. To add packages (like twitter bootstrap or authentication) use `meteor add <packageName>`

We publish on the server:
```
Meteor.publish('PlayersList', function(){
	return PlayersList.find();
});
```

and subscribe on the client for the template where we want to render the data:

```
Meteor.subscribe('PlayersList');
```


###### more code
I define some players list events for upvoting. Events call server side methods here:

```
Template.players_list.events({
  'click .increment': function(e, tmpl){
    Meteor.call("update_player_score", this._id, 5);
  },
  'click .decrement': function(){
    Meteor.call("update_player_score", this._id, -5);
  },
  'click .remove': function(){
    Meteor.call('remove_player', this._id);
  }
});
```
`this` refrences the data context (so the current player in the each loop)

Those methods make mongodb calls. The code is in our server directory and therefore hidden from the client:

```
	'remove_player': function(player_id){
		PlayersList.remove(player_id);
	},
	'update_player_score': function(player_id, toAdd){
		PlayersList.update({_id: player_id}, {$inc: {score: toAdd} });
	}
```


###### That's it
Check out the [source code](https://github.com/connor11528/leaderboard). Deploy your app with `meteor deploy <my-app-name>.meteor.com`

