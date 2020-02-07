---
layout: post
title: Oh hurrow Koa.js
date: "2015-10-23"
category: Javascript
description: "Koa is a Javascript HTTP server framework developed by TJ, the creator of express. It was meant to be the successor to the express npm package as it uses ES6 generators. This tutorial gets started with the framework through building a web server."
author: Connor Leech
published: true
---

Koa is a node.js web framework that is more modern than express. We will build a functional web server in this tutorial 

![](https://camo.githubusercontent.com/674563115c4e0d4e5d99440b916952ad795c498e/68747470733a2f2f646c2e64726f70626f7875736572636f6e74656e742e636f6d2f752f363339363931332f6b6f612f6c6f676f2e706e67)

<iframe src="https://ghbtns.com/github-btn.html?user=jasonshark&repo=koa-starter&type=star&count=true&size=large" frameborder="0" scrolling="0" width="160px" height="30px"></iframe>

<iframe src="https://ghbtns.com/github-btn.html?user=koajs&repo=koa&type=star&count=true&size=large" frameborder="0" scrolling="0" width="160px" height="30px"></iframe>

Koa is a node.js web framework that is more modern than express. We will build a functional web server in this tutorial. Koa takes advantage of es6 features, like [generators](http://davidwalsh.name/es6-generators). Generators do not actually generate anything. Read that article to get a gist (hint `yield` statements matter). I'll be here when you get back.

Welcome back! To use this we need the newest version of node. Check with:

```
$ node -v
v0.12.7
```

Then to run our es6 code with `$ node --harmony server.js`. To not type that --harmony flag all the time add an alias to **~/.bash_profile**: 

```
alias node="node --harmony"
alias nodemon="nodemon --harmony"
```

Harmony means ES6. Nodemon is for [nodemon](https://github.com/remy/nodemon).

### Start project

```
$ mkdir project-name
$ cd project-name && npm init
$ npm install koa koa-route --save
```

Make a simple **server.js**:

```
var koa = require('koa');
var route = require('koa-route');
var app = koa();

// routes
app.use(route.get('/', index));
app.use(route.get('/about', about));

// es6 generators expected as arguments
function *index() {
	// this is a Koa Context
	// A Koa Context encapsulates node’s request and response objects into a single object 
	this.body = "<h1>Hello Koa!</h1>";
}

function *about() {
    console.log('Koa Context (this) has these properties: ');
	console.log(Object.keys(this));
	this.body = "<h2>This is the about route</h2>";
}

app.listen(8008);
console.log('Koa listening on port 8008');
```

Here we see the es6 generators. Route callbacks expect generators, not regular functions. `this` in the generator is special in koa. It contains request and response as well as some other handy bits for web apps. When you hit the /about route you will see this:

```
Koa Context (this) has these properties: 
[ 'request',
  'response',
  'app',
  'req',
  'res',
  'onerror',
  'originalUrl',
  'cookies',
  'accept',
  'state' ]
```


Okay little different, nothing crazy. Next up we want to render some template files. ejs will be our template engine. [co-views](https://github.com/tj/co-views) has some config options for using jade or swig.. whatever that is.

`$ npm install co-views ejs --save`

Then in **server.js** add:

```
var render = views(__dirname + '/views', { ext: 'ejs' });

...

function *index() {
	this.body = yield render('index', {});
}
```

This is similar to express. The empty object is data we can pass to the template engine. `co-body` acts like `body-parser` for incoming input so add that too.

`$ npm install co-body --save`

#### Resources
Koa CRUD app: http://weblogs.asp.net/shijuvarghese/a-simple-crud-demo-with-koa-js

Koa examples templates: https://github.com/koajs/examples/tree/master/templates

In the future we will implement OAuth with [grant](https://github.com/simov/grant) and I will build out a more full featured starter template.

Source code is [AVAILABLE HERE](https://github.com/jasonshark/koa-starter).

<iframe src="https://ghbtns.com/github-btn.html?user=jasonshark&repo=koa-starter&type=star&count=true&size=large" frameborder="0" scrolling="0" width="160px" height="30px"></iframe>
