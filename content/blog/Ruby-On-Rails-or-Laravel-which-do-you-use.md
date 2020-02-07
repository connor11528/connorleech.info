---
layout: post
title: Ruby On Rails or Laravel, which do you use?
date: "2016-11-13"
category: Career
description: "High level blog post comparing the merits of Laravel with Ruby On Rails"
author: Connor Leech
published: true
---

Last week I [wrote
about](/blog/Building-a-feature-complete-bookmarking-app-with-Vue.js-Express-and-Sequelize-ORM/)
how to build a bookmarking application using Sequelize ORM — an application that
allows Javascript to speak SQL. This week we’re going dive in to Ruby On Rails
and Laravel and the pros and cons of each web framework.

![](https://cdn-images-1.medium.com/max/800/1*94Oae3_D1l5fpDPxX_oAQw.jpeg)
<span class="figcaption_hack">Laravel is branded as The PHP Framework for Web Artisans</span>

### Web what?

[Web frameworks](https://en.wikipedia.org/wiki/Web_framework) are libraries of
code with varying degrees of functionality written in a programming language to
carry out common web development tasks. Common tasks include authenticating
users, reading, writing, updating the database, sending emails and interacting
with third party APIs (such as LinkedIn, Facebook and Twitter). Web frameworks
run on the server side of the application and process requests from web clients
(such as browsers, iPhones, toasters etc). Laravel is written in the PHP
programming language. Ruby On Rails is written in the Ruby programming language.

![](https://cdn-images-1.medium.com/max/800/1*xntp5lTnrBD3qJtpz1laFg.jpeg)

<span class="figcaption_hack">Ruby On Rails historically has been one of the most popular frameworks for developers on the web.</span>

### Common Ground

Ruby On Rails and Laravel both operate off of the Model View Controller (MVC)
pattern. This means that both frameworks use their respective programming
languages to model tables in the database, views in HTML and controllers to
process requests. In both frameworks the general flow is user sends request from
web browser (the View), the Controller handles the request and invokes a Model
which then updates the database. Changes are then reflected back in the View via
the Controller.

Additionally, both frameworks rely on Relational Databases. This means the data
is structured similarly to the way Excel spreadsheets are laid out. There are
columns and rows. Structured Query Language (SQL) is what the programming
languages use behind the scenes to issue database commands. Developers use web
frameworks in part because it allows them to write PHP or Ruby instead of raw
SQL queries, making them more productive.

### Background

Ruby On Rails is a much older framework. It was first released by
[DHH](https://medium.com/@dhh) in 2004 for his company Basecamp. Laravel was not
created until 2011 when [Taylor Otwell](https://medium.com/@taylorotwell)
released it to solve problems that plagued existing PHP frameworks. Laravel has
rocketed in popularity in recent years while Rails has decreased in popularity
(according to Google Trends).

![](https://cdn-images-1.medium.com/max/800/1*fix-SvGdj5jhHKliTrLFjw.png)

<span class="figcaption_hack">Laravel has increased in popularity while ROR has declined slightly.</span>

Both frameworks are actively maintained and updated. Laravel and Ruby On Rails
each had their fifth major release (version 5.0.0) this past summer.

> For companies and developers deciding between the two frameworks it is a choice
> between the older, tried and tested software versus the relatively new kid on
the block.

### The Languages

Ruby is decried for being slow. PHP is feared to be messy and unmaintainable.
These are the common critiques of the languages themselves.

![](https://cdn-images-1.medium.com/max/600/1*KEjp_OE32wCfCEzOstiz6g.png)

<span class="figcaption_hack">The infamous Twitter “Fail Whale”</span>

Ruby is not as fast as other programming languages. Twitter experienced issues
with Ruby On Rails in its very early days for processing millions of requests
per second during peak hours. It is important to note though that the language
has increased its speed dramatically from 2004 to 2016. Additionally, there are
Ruby tools out there to increase scalability such as
[Chef](https://www.chef.io/chef/) with AWS and
[Capistrano](http://capistranorb.com/#) (among others) to increase performance.
DHH wrote a great post called [Ruby has been fast enough for 13
years](https://m.signalvnoise.com/ruby-has-been-fast-enough-for-13-years-afff4a54abc7#.1g0ltm38i)
outlining his position.

PHP on the other hand has been around since the very beginning of the web in the
early 1990s. Facebook is built on PHP and has invested heavily in its ongoing
development. PHP has a bad wrap especially among older developers because PHP
frameworks used to be a complete mess. There was not a central way to manage
dependencies. Ruby has gems, Node.js has npm, Python has pip, Java has Mavenand
PHP did not have this system until 2012.

The PHP development landscape changed with the release of
[Composer](https://getcomposer.org/). Developers could now use individual
packages and mix and match instead of relying on one monolithic PHP framework to
handle all of their problems. Composer is required for modern versions of
Laravel.

![](https://cdn-images-1.medium.com/max/800/1*m3Qyep3gp4hRbe_cLCg29w.png)

<span class="figcaption_hack">Package Manager for PHP that changed the game.</span>

### The Community

Both frameworks have surged in popularity because of their accessibility. Ruby
On Rails has [RailsCasts](http://railscasts.com/). Laravel has
[Laracasts](https://laracasts.com/). Both have many free videos but require
subscriptions for unblocked access.

Rails has been around for longer and more has been written about it. Laravel is
growing faster and the community is smaller but more active. There are no
shortage of free resources online for learning either framework.

Which framework are you using? Please feel free to respond in the comments!

<hr>

Originally published [on Medium](https://medium.com/@connorleech/php-laravel-ruby-on-rails-and-web-frameworks-32c1e50cea2d)