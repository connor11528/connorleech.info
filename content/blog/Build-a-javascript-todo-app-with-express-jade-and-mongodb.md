---
layout: post
title: Build a javascript todo app with express, jade and mongodb
date: "2015-10-23"
categories: ["Javascript"]
toc: false
description: "This post goes through the somewhat arduous process of building a todo list application with Node.js, MongoDB and express. Ultimately we deploy the app to the internet with Git and Heroku. In the future I'd recommend some client side JS frameworks or generating an application like this quickly using Laravel."
author: Connor Leech
published: true
---

Hey everyone today we're going to build a super simple todo application using [express 4](http://expressjs.com/4x/api.html). Express is a lightweight server side framework for node.js. It's very popular but not quite as easy to get up and running with a database as something like Rails or Django.

In this tutorial we'll connect an express server to a mongoDB database and serve our html using the jade templating engine. To make talking to MongoDB easier we will use the [mongoose](https://github.com/learnboost/mongoose) npm package. Even though mongodb is a schemaless database (everything is json) mongoose lets us define models and easily chain queries.

[SOURCE CODE](https://github.com/connor11528/express-todo) is on github here

[LIVE DEMO](http://express-mongo-jade-todo.herokuapp.com) hosted by heroku is here

### Set up the project
Unlike Ruby On Rails, express allows you to set up your file structure any way you want. This was confusing to me when looking at other people's code because I wasn't sure what the best way to organize my project was. Here's the directory structure I'm going to use today:

```
express-todo
  |- config
  |- public
    |- javascripts
    |- stylesheets
    |- images
  |- routes
  |- views
  .gitignore
  package.json
  server.js
  README.md
```

**public -** this folder will hold files that everyone can see when they visit our page with a web browser. Here's where we'll put client side javascript and css.

**config -** here we're going to have everything to do with configuring our environments and managing our database

**routes -** this is where we'll have the server side application logic. If you're coming from a rails background you might think of these as controllers

**views -** our jade templates live here

**package.json -** defines all packages (like express and mongoose) that our application depends on

**server.js -** this is the main executable file for everything. To run our app we'll run `node server` or `node server.js` from the command line (terminal on mac). If something doesn't work, start here.

### Build a server
Okay so now let's get coding! We're going to start but running up our **server.js** file to listen for hits to a specific port on our computers. With this file we'll run a local web server on our computer. Later we'll deploy this as a real web server for the interwebs.
```
var express = require('express');
var path = require('path');
var favicon = require('serve-favicon');
var logger = require('morgan');
var cookieParser = require('cookie-parser');
var bodyParser = require('body-parser');
var port = 3000;

var app = express();

// view engine setup
app.set('views', path.join(__dirname, 'views'));
app.set('view engine', 'jade');

//app.use(favicon(__dirname + '/public/favicon.ico'));
app.use(logger('dev'));
app.use(bodyParser.json());
app.use(bodyParser.urlencoded());
app.use(cookieParser());
// telling Express to serve static objects from /public 
app.use(express.static(path.join(__dirname, 'public')));

// Start server
app.listen(port, function(){
  console.log('Server listening on port ' + port)
});
```

First we tell express where to find our templates (ie the views directory) and then tell express to use jade as our template engine. [body-parser](https://www.npmjs.com/package/body-parser) is for handling form inputs. Cookies we won't use but I included anyways. The next line tells express where to serve static assets from so all our server code isn't visible to everybody.

You can check this code works by running `node server` in the command line. You should get "Server is listening on port 3000". Then check `localhost:3000` in your web browser and you'll see an error

"Cannot GET /"

This is because we haven't defined any routes.

### Define our routes
Routes are essentially urls for your website. So when you went to `localhost:3000` you visited the base root `/`. For our website we'll want routes for viewing, creating and editing todos. So modify **server.js** after the express.static line and before app.listen.

```
// Routes
var main = require('./routes/main');
var todo = require('./routes/todo');
var todoRouter = express.Router();
app.use('/todos', todoRouter);

app.get('/', main.index);
todoRouter.get('/', todo.all);
todoRouter.post('/create', todo.create);
todoRouter.post('/destroy/:id', todo.destroy);
todoRouter.post('/edit/:id', todo.edit);
```

Also create two files, **main.js** and **todo.js** in the routes folder. Before jumping in to those files let's look at the new express routes we defined. The user can visit our homepage `/`, they'll be able to see all todos at `/todos`, create todos by sending a post request to `/todos/create` and delete todos by sending a delete request to `/todos/destroy/:id`. `:id` is will be the unique database id provided to us by mongodb. Finally they'll be able to update todos by sending a put request to `/todos/edit/:id` and view individual todos at `/todos/:id`.

Each of these scenarios will be handled with a function that takes two parameters -- request and response.

Define the todo handler functions in **routes/todo.js** with some empty content for now:

```
module.exports = {
    all: function(req, res){
        res.send('All todos')
    },
    viewOne: function(req, res){
        console.log('Viewing ' + req.params.id);
    },
    create: function(req, res){
        console.log('Todo created')
    },
    destroy: function(req, res){
        console.log('Todo deleted')
    },
    edit: function(req, res){
        console.log('Todo ' + req.params.id + ' updated')
    }
};
```

So in this file we have an object with keys and handler functions as values. `module.exports` makes the object in this file usable in our server.js file. `req.params.id` takes the value directly from the url bar. Try visiting `localhost:3000/todos/blahblahblah` in your web browser. The console.log messages will log out in your terminal and you will see "Viewing blahblahblah".

### Getting jaded

Let's get the homepage setup. In our server.js `app.get('/', main.index);` this line handles our homepage. Define the controller function in routes/main.js:

```
module.exports = {
	index: function(req, res) {
		res.render('main', { title: 'Express Todo' });
	}
};
```

`res.render` renders a view. We already told express our views are in the `views` folder and that we're using jade templating. Create a file **main.jade**:
```
extends layout

block content
	h1= title
	p Welcome to #{title}
    a(href='/todos').btn.btn-success.btn-lg View all todos
```

And also create a **layout.jade** in the views folder:
```
doctype html
html
	title Express Todo
    link(rel="stylesheet", href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.1/css/bootstrap.min.css")
	link(rel="stylesheet", href="/stylesheets/main.css")
body
	block content
```

Here we include twitter bootstrap cdn link for styling. Click the big green button and you'll navigate to `/todos`. Be careful not to mix tabs and spaces in your jade code. You have to use one or the other or Mrs. Jade becomes angry

### Add MongoDB

Now that we have our server up and serving up content, let's connect it to a local database. First add mongoose as a dependency. So from the terminal:

`$ npm install mongoose --save`

Also check that you have mongodb installed with `$ mongo --version`. Open a brand new terminal and run 

`$ mongod`

This wakes MongoDB up and allows it to operate. In order to connect to a local mongodb database, mongod must be running.

In another terminal window enter `$ mongo` we're going to create a local mongodb database:

```
$ mongo
MongoDB shell version: 2.6.4
connecting to: test
Server has startup warnings: 
2014-12-24T10:30:36.556-0800 [initandlisten] 
2014-12-24T10:30:36.556-0800 [initandlisten] ** WARNING: soft rlimits too low. Number of files is 256, should be at least 1000
```
In this REPL type `use express-todo` to create our local database. To see all the databases you have type `show dbs`. To quit the mongo terminal press Ctrl + C

#### Define data model and connect

Even though mongodb is schemaless, to model our data with mongoose we still need to define models. Create a new folder inside config called **models** and add a new file called **Todo.js**. We'll then define what our todo data will look like with a schema:

```
var mongoose = require('mongoose'),
	Schema = mongoose.Schema;

// todo model
var todoSchema = new Schema({
    content: String,
    completed: { type: Boolean, default: false },
    updated_at: { type: Date, default: Date.now }
});

mongoose.model('Todo', todoSchema);
```

Next in **server.js** add this code above the code for the routes:

```
require('./config/models/Todo');
mongoose.connect('mongodb://localhost/express-todo', function(){
	console.log('connected to database!')
});
```

This registers our todo schema and connects to the local mongodb database.

### Show Todos
Even though our database is empty let's query it. Update the code in `routes/todo.js`:

```
var mongoose = require('mongoose'),
    Todo = mongoose.model('Todo');

module.exports = {
    all: function(req, res){
        Todo.find({}, function(err, todos){
            if(err) res.send(err);
            res.json(todos);
        })
    }
...
```
Here we include the Todo model we registered earlier, query it for all entries (the first parameter is optional here) and then respond with JSON to the web page.

If you visit `localhost:3000/todos` you should see an empty array on the page

### Creating todos
Create a view file called **todos.jade** that we'll display all the todos on.

```
extends layout

block content
	h1.text-center All Todos
	ul
		for todo in todos
			li #{todo.content}
	form(method='post', action='/todos/create')
		.input-group
			input.form-control(type='text', placeholder='What do you have to do?', name='content')
			span.input-group-btn
				input.btn.btn-success(type='submit', value='Add Todo')
```
Here we have an unordered list to display the todos and a form that hits our server with a post request to add one.

Let's define that route that the form hits. Open `routes/todo.js`. Here is the code for creating a todo and then reloading the page.

```
...
create: function(req, res){
    var todoContent = req.body.content;
    // create todo
    Todo.create({ content: todoContent }, function(err, todo){
        if(err) res.render('error', { error: 'Error creating your todo :('})
        // reload collection
        res.redirect('/todos');
    });
},
...
```
Congrats! You now have a javascript server that can talk to a database and render the information in a web browser

### Deleting todos

First update **todos.jade**

```
block content
	h1.text-center All Todos
	.panel.panel-default
		for todo in todos
			.panel-body
				a(href='/todos/#{todo._id}') #{todo.content}
				form(action='/todos/destroy/#{todo._id}', method='post')
					input.btn.btn-danger.pull-right(type='submit', value='Delete')
```
For this we submit a form when pressing the button. The styling is not perfect but you can mess with that however you like.

Here's the code to delete a todo and reload the page:

```
destroy: function(req, res){
    var id = req.params.id;

    Todo.findByIdAndRemove(id, function(err, todo){
        if(err) res.render('error', { error: 'Error deleting todo'});
        res.redirect('/todos');
    });
},
```

### View and update todos
I'd like for each todo to have its own page and be able to edit the todo on that page. Create a **todo.jade** file in the views directory:

```
extends layout

block content
	form(method='post', action='/todos/edit/#{todo._id}')
		h1
			input(value='#{todo.content}', name='content')
			input(type='submit', value='Update').btn.btn-primary.btn-lg
			a(href='/todos').btn.btn-default.btn-lg Back
```
This extends the layout again and has a form for updating and also a back button. Here are the methods for viewing the page and handling the update request. (We redirect them back to the full list after update is complete.)

```
viewOne: function(req, res){
    Todo.find({ _id: req.params.id }, function(err, todo){
        res.render('todo', { todo: todo[0] })
    });
},
edit: function(req, res){
        Todo.findOneAndUpdate({ _id: req.params.id }, {content: req.body.content}, function(err, todo){
            res.redirect('/todos');
        });
    }
```
Now we have a page for showing all todos and also each todo has its own page with its own unique url. Also we can add, delete and update all of our todos.

### Refactor

Our code thus far won't work in production because we're reading from a local database. Also the server.js file has a lot of stuff thrown in there and does not need to be that heavy. Let's clean it up.

New **server.js** file:

```
var express = require('express');
var app = express();

// Environments
var env = process.env.NODE_ENV || 'development';
var envConfig = require('./config/env')[env];

// Express configuration
require('./config/config')(app, envConfig);

// Database
require('./config/database')(envConfig)

// Routes
require('./config/routes')(app);

// Start server
app.listen(envConfig.port, function(){
  console.log('Server listening on port ' + envConfig.port);
});
```

So the first thing I did was create an `env.js` file in the config folder. That exports an object with a key for each environment (in our case production and development).

```
var path = require('path');
var rootPath = path.normalize(__dirname + '/../'); // normalizes to base path

module.exports = {
	development: {
		rootPath: rootPath,
		database: 'mongodb://localhost/express-todo',
		port: process.env.PORT || 3000
	},
	production: {
		rootPath: rootPath,
		database: '',
		port: process.env.PORT || 80
	}
};
```

Next up I moved the express config into a file called `config.js` in the config directory:

```
var express = require('express');
var logger = require('morgan');
var path = require('path');
var cookieParser = require('cookie-parser');
var bodyParser = require('body-parser');
var favicon = require('serve-favicon');

module.exports = function(app, envConfig){
	// view engine setup
	app.set('views', path.join(envConfig.rootPath, 'views'));
	app.set('view engine', 'jade');

	//app.use(favicon(envConfig.rootPath + '/public/favicon.ico'));
	app.use(logger('dev'));
	app.use(bodyParser.json());
	app.use(bodyParser.urlencoded());
	app.use(cookieParser());
	// telling Express to serve static objects from the /public/ dir, but make it seem like the top level
	app.use(express.static(path.join(envConfig.rootPath, 'public')));
};
```

Then moved the database connection login into `database.js` in the config folder:

```
var mongoose = require('mongoose');

module.exports = function(envConfig){
	// register models
	require('./models/Todo');

	// connect to database
	mongoose.connect(envConfig.database, function(){
		console.log('connected to database!')
	});
};
```

Then moved all the routes into `routes.js` also in the config folder:

```
var express = require('express');

module.exports = function(app){
	// register route controllers
	var main = require('../routes/main');
	var todo = require('../routes/todo');
	var todoRouter = express.Router();
	app.use('/todos', todoRouter);

	app.get('/', main.index);
	todoRouter.get('/', todo.all);
	todoRouter.get('/:id', todo.viewOne);
	todoRouter.post('/create', todo.create);
	todoRouter.post('/destroy/:id', todo.destroy);
	todoRouter.post('/edit/:id', todo.edit);
};
```

Check that everything works locally. Also if anyone has tips for a more elegant way to manage environments in express apps let me know

### Deploy to Heroku

We're going to use heroku and git for hosting and deploying our node.js app. First create a `.gitignore` file in the root and include node_modules:

```
node_modules/
```
This will ignore checking in npm stuff. Heroku will do that anyways. Next add a git file, create one and add your work.

```
$ git init
$ git add -A
$ git commit -m 'initial commit'
```
Next up we're going to create a new application using heroku's command line tool.
```
$ heroku create
```
This will give your app some nonsensical placeholder name. To change it follow this syntax:

```
$ heroku apps:rename newname
```
Next add a Procfile to your app. So in the root of your project (where server.js is) add a file named `Procfile`. In there write:

```
web: node server.js
```
This declares that there's a web process and to start it use the node server command. 

Next set the environment to "production"
```
$ heroku config:set NODE_ENV=production
```
This will use the production config object we defined in `config/env.js`. Next we're going to update the database key in that object. Head over to [MongoLab](https://mongolab.com/) and create a database. We're going to connect to this database via a URI. There's also a [heroku addon](https://addons.heroku.com/mongolab) you can use but it requires a credit card.

Update the production config object so it looks similar to this:

```
production: {
		rootPath: rootPath,
		database: 'mongodb://jasonshark:multivision@ds037478.mongolab.com:37478/multivision',
		port: process.env.PORT || 80
}
```

Finally deploy the app to heroku:

```
$ git push heroku master
```

Check out the [live demo](http://express-mongo-jade-todo.herokuapp.com) and [source code](https://github.com/connor11528/express-todo)