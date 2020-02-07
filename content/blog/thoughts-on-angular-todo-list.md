---
layout: post
title: thoughts on angular todo list
date: "2015-10-22"
category: Javascript
lede: "Angular 1.x has some tricks. This post looks at the source code for a client side todo list app"
author: Connor Leech
published: true
---

Complaints about angular.js after playing around extensively with React and Meteor

I'm revisiting angular after not using it for a while. To get started again with my class @ RocketU we made a todo list. My code is here:

[JS Fiddle Todo list](http://jsfiddle.net/9cxpha19/)

```
<div ng-app='TodoList' ng-controller='TodoCtrl as tc'>
<h1>To list</h1>

<input type='text' placeholder='Add a todo...' ng-model='tc.newTodo' ng-keypress="tc.checkEnter($event)">   
    <button ng-click='tc.addTodo($event)'>Add todo</button>
<ul>
    <li ng-repeat='todo in tc.todos track by $index' ng-class='{done: todo.completed}'>
        <input type='checkbox' ng-model='todo.completed'>
        {{todo.name}} - <span ng-click='tc.remove($index)'>X</span>
    </li>
</ul>
```

First the markup seems real dirty. There are all sorts of extra the developer should be clued in about like `ng-class`, `ng-keypress`, passing in `$index` and `$event`. I like how I can attach `ng-model` to the checkbox to make it automatically checked. I had trouble updating the data and the state of the checkbox using backbone. Angular's two way data binding is for sure awesome. 

```
angular.module('TodoList', [])
.controller('TodoCtrl', function(){
    var self = this;
    self.todos = [{name: 'Learn angular', completed: true}, 
                  {name: 'Learn React', completed: false}];
    
    self.addTodo = function(e){
        self.todos.push({name: self.newTodo, completed: false});
        self.newTodo = '';
    };
    
    self.checkEnter = function(e){
        if(e.keyCode === 13){
            self.addTodo();
        }
    };
    
    self.remove = function(index){
        self.todos.splice(index, 1);
    };
});
```

We used the new `controller as` syntx, which is less of a pain than I was expecting. As long as we use `var self` we don't run into any problems. I bet when Angular 2.0 and ES6 come out we won't have to worry about the scope of this and whatnot. Looking forward to diving back into angular directives!

Soon I'll also post a bit on how to build these simple apps. I like the declarative nature of react. It feels like a whole lot less magic. To infinity and beyond!