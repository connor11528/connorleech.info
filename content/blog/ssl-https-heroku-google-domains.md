---
layout: post
title: HTTPS/SSL on Heroku with Google Domains as DNS provider
date: "2017-10-18"
category: Cloud
lede: "I needed HTTPS for Stripe to work. Here's the saga as it unfolded. Add an SSL certficate with Let's Encrypt, configure Heroku and set up Google Domains. Oh my"
author: Connor Leech
published: true
---

![](https://cdn-images-1.medium.com/max/800/1*0NcMrplkohfvuKIJkAqmqw.png)

This might be a bit of a niche share but it is something I struggled with recently. I have a Node.js application hosted on Heroku that I wanted deployed to `https://www.mydomain.com`. I bought my domain name through Google Domains as the DNS provider.

This means that even if the user puts in `http://mydomain.com` or `http://www.mydomain.com` it all needs to end up on `https://www.mydomain.com`. I’m not trying to belabor the point and it may seem obvious but it was non-trivial for me to set up the first time.

**tl;dr** — the website is hosted live here: http://www.linemansmilestones.com/ (it’ll forward the URL to the https version of the site)

...

**Read the full blog post [on Medium](https://medium.com/@connorleech/https-ssl-on-heroku-with-google-domains-as-dns-provider-c55c438556c6)**