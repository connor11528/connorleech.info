---
layout: post
title: ES6 arrow functions and this
date: "2015-10-23"
category: Javascript
description: "Intro to ES6 arrow functions"
author: Connor Leech
published: true
---

ES6 is the upcoming standard for javascript. You can use ES6 features today by including babel.js or the jsx transformer in your html file. React uses JSX and the JSX transformer library to take advantage of ES6 features and generate components. Angular 2.0 uses typescript to implement [select](http://stackoverflow.com/a/22386542/2031033) ES6 features. Moving forward React and Angular will commonly use arrow functions. If you are familiar with coffeescript you already know about them. Below is a rundown of arrow functions and this.

To run the ES6 version of javascript in chrome download [Scratch JS](https://chrome.google.com/webstore/detail/scratch-js/alploljligeomonipppgaahpkenfnfkn?hl=en-US) and a new tab will be added to your console.

<b>arrow functions</b> - at first arrow functions seemed obvious.
```
// Expression bodies
var odds = evens.map(v => v + 1);
var nums = evens.map((v, i) => v + i);

// Statement bodies
nums.forEach(v => {
  if (v % 5 === 0)
    fives.push(v);
});
```
You don't need return statements. The first letter is the parameter. If there are multiple parameters use parenthesis. The syntax change makes writing javascript especially more clean when writing this. 

<b>arrow functions and this</b> - Unlike functions, arrows share the same lexical this as their surrounding code.

```
// ES6 Code
var bob = {
  name: "Bob",
  friends: ['Jane', 'Mary', 'Dan'],
  printFriends() {
    this.friends.forEach(f =>
      console.log(this.name + " knows " + f));
  }
}

bob.printFriends();

// Vanilla.js
var joe = {
  name: "Joe",
  friends: ['Bob', 'Lue', 'Shaniqua'],
  printFriends: function() {
    this.friends.forEach(function(f){
      console.log(this); // window (global this)
      console.log(this.name + " knows " + f);
    });
  }
}

joe.printFriends();
```

this.name in vanilla JS is an empty string. Type window.name in the browser console, you'll get empty string. this in the vanilla JS refers to window, not the joe object. The ES6 code will work as expected and print "Bob know Jane" etc. Using arrow functions this.friends and this.name share the same this value.

In order to make the ES5 vanilla JS code to work we have to store the value of this to a variable so we can access the name property of the parent (in this case the joe object).
```
var joe = {
  name: "Joe",
  friends: ['Bob', 'Lue', 'Shaniqua'],
  printFriends: function(){
    var _this = this;  // refers to joe object
    this.friends.forEach(function(f){
      console.log(_this.name + " knows " + f)
    });
  }
}
```

Now we get the expected behavior. In summary, this within arrow functions is the same as this in the parent. No new scope is created with arrow functions.

<b>Resource:</b>
[babel website](https://babeljs.io/docs/learn-es6/) -  learn es6