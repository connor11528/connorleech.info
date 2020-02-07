---
layout: post
title: Create a Wordpress site from the command line with wp-cli
date: "2018-02-20"
category: PHP
lede: "Wordpress is super prevalent but WP development has always been a bit of a mystery to me. This tutorial goes through running a Wordpress site locally on your machine."
author: Connor Leech
published: true
---

The first step is to install the wordpress command line tool. The below commands will download the package, grant the appropriate permissions for using the files and move the package to a place in our computer to a place where we can conveniently use it.

```
$ curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar
$ chmod +x wp-cli.phar
$ sudo mv wp-cli.phar /usr/local/bin/wp
$ wp
$ mkdir wordpress-local-site && cd wordpress-local-site
$ wp core download
$ wp server
$ mysql -uroot -p
> create database wordpress;
```

Full post is [on Medium](https://medium.com/@connorleech/create-a-wordpress-site-from-the-command-line-and-run-it-locally-13db3f996519)
