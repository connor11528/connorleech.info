---
layout: post
title: Create a comment list component out of React.js
date: "2015-10-23"
category: Javascript
lede: "While working as an Angular 1.x developer I decided to look into React.js because it was the new hotness. This guide walks through creating a CommentList and Comment component that fetches data via AJAX."
author: Connor Leech
published: true
---

Clone the [starter code](https://github.com/connor11528/react-comment-list/tree/f7f7b0451edc008b25817c6fedb44b42001613ee) or view the [finished code](https://github.com/connor11528/react-comment-list/tree/master).

### Create components
commit code: https://github.com/connor11528/react-comment-list/tree/fd7504266de7266fa16b9286b57eb39a6ba00287
```
class CommentList extends React.Component {
	render(){
		return (
			<div className='comment-list'>
				CommentList
			</div>
		);
	}
}

class CommentForm extends React.Component {
	render(){
		return (
			<div className='comment-form'>
				CommentForm
			</div>
		);
	}
}

class CommentBox extends React.Component {
	render(){
		return (
			<div className='comment-box'>
				<h1>Comments</h1>
				<CommentList />
				<CommentForm />
			</div>
		);
	}
}

// add to DOM
React.render(
	<CommentBox/>,
	document.getElementById('content')
);
```

React components have a render method that returns regular HTML. Most of our html and javascript will be bound together in this JSX file. JSX compiles to raw javascript and uses many ES6 features like "class" and "extends" and arrow functions. We could do all of this in vanillaJS.

Fire up our server with `npm install` and then `npm start`. This will launch the express server that is chilling in the "bin" directory. (More magic to come from that file later).

### Render comments to screen
commit code: https://github.com/jasonshark/react-comment-list/blob/9ed1325f4d55fa467fdc435db72dac4d2e196551/app/app.jsx

Create a comment component:
```
class Comment extends React.Component {
	render(){
		return (
			<div className='comment'>
				<div className='comment-body'>
					{this.props.children}
				</div>
				<div className='comment-author'>
					{this.props.author}
				</div>
			</div>
		)
	}
}
```

Now we want to render a list of these individual comments. First we'll pass an array of normal javascript objects to the commentlist component: `<CommentList comments={comments} />` where
```
var comments = [
	{ id: '1', author: 'Connor Leech', text: 'comment from the array'},
	{ id: '2', author: 'Long John Silver', text: 'arrrrrrr'}
];
```

Next in the comment list component we'll build up an html string in a variable called commentNodes and render it:

```
class CommentList extends React.Component {
	render(){
		console.log(this.props.comments)
		var commentNodes = this.props.comments
			.map(function(comment){
				return (
					<Comment author={comment.author} key={comment.id}>
						{comment.text}
					</Comment>
				);
			});
		return (
			<div className='comment-list'>
				{commentNodes}
			</div>
		);
	}
}
```
`this.props.comments` refers to the comments array we passed to the component. In React `{}` denotes raw javascript (escapes the HTML).


### Add reactive state
commit code: https://github.com/connor11528/react-comment-list/blob/8d4c07a2c4b13465e8fb1806ff4a9720f9cd7970/app/app.jsx

Now we're going to pass the comments array to the comments box. The advantage of doing this is that when the state changes the view will update very quickly, without page refresh.

```
class CommentBox extends React.Component {
	// constructor function recieves object of attributes + values
	constructor(props){
		super();
		this.state = {
			comments: props.comments
		}
	}
	render(){
		return (
			<div className='comment-box'>
				<h1>Comments</h1>
				<CommentList comments={this.state.comments} />
				<CommentForm />
			</div>
		);
	}
}
```

The component constructor function sets the initial values for the components. The CommentList receives its value from the state and the state is set through a property:

```
commentBox = React.render(
	<CommentBox comments={comments}/>,
	document.getElementById('content')
);
```
You can play with `commentBox` and the setState() method in the browser console to watch updates 


React re-renders entire DOM tree in memory when there is a state change. The rendering process that exists in memory is called the virtual DOM. The virtual dom does not get rendered to the page. Instead React does a diff between the DOM tree and the Virtual DOM going node by node, looking for differences. Then there is an array of instructions to convert the DOM with the minimum number of changes.


Finally make an AJAX request and use componentDidMount: https://github.com/connor11528/react-comment-list/commit/90e0cdd2a58659ffaebcb7ede9ebd03d713cc4fd

Then add comment (the server will write to comments.json): https://github.com/connor11528/react-comment-list/commit/ef3852a7908028e2a9418412cb0fbfe0ee3e85dc

Hope this was helpful. Any questions about this tutorial you can [tweet me](http://twitter.com/connor11528)