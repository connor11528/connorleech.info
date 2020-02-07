---
layout: post
title: Intro to React
date: "2015-10-24"
category: Javascript
description: "General introduction into how React.js can work as a view layer"
author: Connor Leech
published: true
---

Oh hurrow React.js

[SOURCE CODE](https://github.com/connor11528/intro-to-react)

<iframe src="https://ghbtns.com/github-btn.html?user=connor11528&repo=intro-to-react&type=fork&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>

<iframe src="https://ghbtns.com/github-btn.html?user=connor11528&repo=intro-to-react&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>

In this tutorial we're going to go through the basics of React. We will render a list of posts that we return via a ajax query. This will not be fully functional as I'm also a beginner to react. I'll cover what I know.

#### First off, why react?
React exists to update apps and websites in response  to constant changes in data. React is only the view layer. So if you want to make an http call or model data you need other tools. For our trivial example jQuery is sufficient. To use react for all the capabilities of angular.js you need other tools. Facebook advises to use the Flux pattern. This is not a library but instead an idea. I found that incredibly unhelpful. I think it is best to not take flux to seriously. If you start coding consistently in react best practices will become apparent. Right now many things are being forged out. Angular has a head start as far as best practices are concerned, but that could all go out the window when Angular 2.0 comes out. This will be a gentle introduction to what this whole react thing *is*.

React is only the view. Think of it as a substitute for html, but more complicated and made out of javascript. It is common to see react used with Buildpack or browserify, using npm modules on the client side. It is also common to see react written on the server side. React is good because it keeps javascript and html coupled tightly together. JSX is a syntax that makes it possible to write html in javascript code. React compiles JSX into HTML and keeps track of updates through a virtual DOM.

###### The Virtual DOM
React is so fast because it never talks to the DOM directly. React maintains a fast in-memory representation of the DOM. `render` methods return a description of the DOM, and React can diff this description with the in-memory representation to compute the fastest way to update the browser.

#### back to basics
All you need is html and a javascript file. Set up an **index.html**:

```
<body>
    <div id="content">
    </div>

    <!-- scripts -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/2.1.1/jquery.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/react/0.13.0/react.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/react/0.13.0/JSXTransformer.js"></script>
    
    <script type="text/jsx;harmony=true" src="js/app.jsx"></script>
  </body>
```

For this we include react.js, the JSXTransformer, jQuery and a div where our entire application will render. The JSXTransformer library converts JSX to raw javascript. You can write react out of normal, ES5 javascript but it is more work. JSX will make our lives with React easier. The whole point of react is to couple our html and javascript tightly. It makes sense to use the transformer. You can use [ES6 features](http://es6katas.org/?utm_source=javascriptweekly&utm_medium=email) like [arrow functions](http://connorleech.ghost.io/es6-features/) with React and JSX. 

Then in **js/app.jsx** we'll render content to the screen.
```
React.render(<h1>Hello React</h1>, document.getElementById('content'));
```
This is a common pattern, but instead of rendering html to the screen like this, most apps will render a component.


###### First component
Everything you build in React will be a component. React is more simple than angular because it does less. Everything in react is a component. React is only the view.

```
var App = React.createClass({
    render: function(){
        return (
            <h1>Hello, world!</h1>
        );
    }
});

React.render(<App />, document.getElementById('content'));
```

Think of components as simple functions that take in "props" and "state" and render HTML. The component can only return one html node, in this case that's an `<h1>`. JSX is similar to normal HTML but not the same.


#### Hello props

```
var App = React.createClass({
    render: function(){
        return (
            <h1>Hello {this.props.title}!</h1>
        );
    }
});

React.render(<App title='Title Property' />, document.getElementById('content'));
```

#### Hello states, what up AJAX

React thinks of UIs as simple state machines. By thinking of a UI as being in various states and rendering those states, it's easy to keep your UI consistent.

![what up Iliad](http://static.comicvine.com/uploads/original/12/127594/4621261-6047962655-u-mEl.gif)

In React, you simply update a component's state, and then render a new UI based on this new state. React takes care of updating the DOM for you in the most efficient way.

```
var App = React.createClass({

	getInitialState: function(){
		return {
			posts: []
		};
	},
	componentWillMount: function(){
		var self = this;
		$.get('/api/posts', function(posts){
			self.setState({posts: posts})
		});
	},
	render: function(){
		return (
			<div>
				<h1>All Posts</h1>
				<List posts={this.state.posts}/>
			</div>
		);
	}
});

var List = React.createClass({
	render: function(){
		return (
			<ul>
				{
					this.props.posts.map(function(post){
						return <li key={post.id}>{post.title}</li>
					})
				}
			</ul>
		);

	}
});

React.render(<App />, document.getElementById('content'));
```

This Here we call to get "posts" from a backend and render a list to the page. The key is necessary for react when rendering a list of elements. We use jQuery here for its ajax functionality. In React, an owner is the component that sets the props of other components. So through the owner component `<App />` we set the state and pass that as a property to the List function. In React, data flows from owner to owned component through props. is the magic of JSX with React.

![](http://media.giphy.com/media/eaJyDondNeLHq/giphy.gif)

#### More on state
`setState(data, callback)` informs React of data change. This method merges data into `this.state` and re-renders the component. When the component finishes re-rendering, the optional callback is called. Most of the time you'll never need to provide a callback since React will take care of keeping your UI up-to-date for you. More magic in the [docs](https://facebook.github.io/react/docs/interactivity-and-dynamic-uis.html).

#### Component Lifecycle
Components have a [lifecycle](https://facebook.github.io/react/docs/component-specs.html) for when they are being added, changed or removed from the DOM. For each stage of the lifecycle we can call a function. Here are some prominent lifecycle states:

**Mounting**: `componentWillMount` Invoked once before being added to the DOM

**Updating**: `componentDidMount` Invoked once when component is added to the DOM. Ajax requests and interactions with other libraries go in this method.

**Unmounting**: `componentWillUnmount` Invoked before a component is removed from the DOM.

#### Resources

Facebook's [React Getting Started Guide](https://facebook.github.io/react/docs/why-react.html)

[Eventedmind](https://www.eventedmind.com/) is good, costs money.

#### Conclusion

In React I like how the html and javascript are closely coupled. I think the simplicity of only dealing with props and state is also cool. In a way it is nice to have limited functionality with React, that way I can use whatever other libraries I want. With coding React I also feel I'm getting better at using javascript, npm modules and new ES6. What I learn with React does not feel React specific.

On the other hand, React has a steep learning curve and does not provide all the functionality I want. I am not familiar with Backbone and implementing thing the "Flux way" might be another headache. Routing is another concern. Maybe in the future I will look into something like [react-router](https://github.com/rackt/react-router). I am excited for the React ecosystem to mature and for the Angular 2.0 release to implement features pioneered by React. Everyone can win!

To dig in more, check the [SOURCE CODE](https://github.com/connor11528/intro-to-react). (Pull requests are welcome)

<iframe src="https://ghbtns.com/github-btn.html?user=connor11528&repo=intro-to-react&type=fork&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>

<iframe src="https://ghbtns.com/github-btn.html?user=connor11528&repo=intro-to-react&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>