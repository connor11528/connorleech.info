---
layout: post
title: Building a feature complete bookmarking app with Vue.js, Express and Sequelize ORM
date: "2016-11-02"
categories: ["Javascript"]
toc: false
lede: "This tutorial walks through building a Node.js, MySQL and front-end framework (Vue.js) application. Client side code is bundled with webpack."
author: Connor Leech
published: true
---

![](http://i.imgur.com/QSTJPpP.png)

#### **Summary**

This tutorial walks through building a Node.js, MySQL and front-end framework
(Vue.js) application. Client side code is bundled with webpack.

**Here is the github link for viewing the application source code:
**[https://github.com/connor11528/sequelize-bookmarks](https://github.com/connor11528/sequelize-bookmarks)

#### Purpose

The reason I am writing this is showcase how SQL, Node.js and modern client-side
Javascript frameworks (Vue.js with Webpack) can all play nicely together. Over
the course of my development career I have moved from PHP to Ruby to Node.js.
For all of last year MEAN stack (MongoDB, Express, Angular, Node.js) development
has been very popular. Now though, Angular 1.x is showing its age. Express is
still the dominant Node.js web framework, but more robust solutions are being
built (such as [Trails.js](https://github.com/trailsjs/trails) and
[Adonis.js](http://www.adonisjs.com/)) that leverage the power of SQL. I thought
it was about time to understand how to work with a SQL database and Javascript,
specifically Node.js.

#### The App

This application is capable of Creating, Reading, Updating and Deleting (CRUD)
Authors and Books from a MySQL database. Each Book belongs to an Author and a
each Author has many books. For a great review of how SQL database associations
work [review here](http://guides.rubyonrails.org/association_basics.html).

#### The Stack

I followed [John Kariuki](https://github.com/johnkariuki)’s excellent tutorial
for the backend code. (If you have not seen [scotch.io
](https://scotch.io/)check it out for great development resources!)

There was no part 2 for this tutorial series and I was sick of writing
Angular.js 1.x code. I have played with React.js but quickly became confused as
to the best way to manage async state across components. I knew Vue.js was
popular in the [Laravel](https://laravel.com/) community so decided to check it
out.

#### Vue.js

I quickly new that this was my new front end framework. Vue.js is cleaner and
easier to reason about than Angular.js 1.x and is easier to learn than React.js.
It comes with many of the features you expect, and for more complex solutions
the core team maintains [Vuex](https://github.com/vuejs/vuex).

For learning Vue.js I suggest Jeffrey Way’s [Learning Vue 1.0: Step By
Step](https://laracasts.com/series/learning-vue-step-by-step)

The most difficult part was getting the app setup. I could have split the
backend API from the frontend into two repositiories (as that is recommended by
the core team) but whatever YOLO.

![](https://cdn-images-1.medium.com/max/800/1*8mHeIgOJ1Li5QCGimqr7mA.gif)

#### Getting Set Up

I used the Vue command line tool:
[https://github.com/vuejs/vue-cli](https://github.com/vuejs/vue-cli)

I highly recommend using [Vueify](https://github.com/vuejs/vueify). This tool is
responsible for those funky **.vue** files in the client side directory. This
allows you to put all the code related to a component in a single file. Each
file will have it’s own Vue instance which acts like scope in angular. It is
easy to reason about as it is essentially an object with about four properties.

For routing: [vue-router](https://github.com/vuejs/vue-router)

For AJAX requests: [vue-resource](https://github.com/yyx990803/vue-resource)

All of these are maintained by the Vue Github organization.

For some extra awesome typeahead functionality I used
[vue-strap](https://github.com/yuche/vue-strap).

#### Conclusion

This app can me deployed to heroku by messing with configuration variables and
installing some SQL addons. I am not going to do that.

![](https://cdn-images-1.medium.com/max/800/1*X4sRREOY3EjwSTuaXM8mSA.gif)

If you need a bookmark application use Goodreads! On a brighter note though, the
[source code](https://github.com/connor11528/sequelize-bookmarks) for this
application will get you up and running with Vue.js, Sequelize, Node.js and
Webpack.

If you have any questions [hit me up on
Twitter](https://twitter.com/Connor11528).

If you enjoyed this breakdown tell your friends to tell their friends.

Thank you!

<hr>

Originally published [on Medium](https://medium.com/@connorleech/building-a-feature-complete-bookmarking-app-with-vue-js-express-and-sequelize-orm-b36506ebcb4c)
