---
layout: post
title: q1 review and goal setting
date: "2020-04-23"
categories: ["goals", "quarterly review"]
toc: false
description: ""
author: Connor Leech
published: true
---

I remember being in my Dad’s sail loft logging in to Netscape Navigator. His internet still had the dial tones but I remember it being faster than my school’s computers and it gave me something fun to do while my Dad finished working. I didn’t really think back to all my computer experiences until after I learned to code after college. Once something’s in motion it’s easy to see how it got there. Steve Jobs famously said that you can connect the dots looking backwards but not looking forwards, no one can. Now that I’ve learned to code and have been doing it professionally for a couple years I can look back and think to myself “wow I’ve always been into computers”. I played Red Alert and Age of Empires and downloaded music from Kazaa and Napster. In 5th grade we had a great time changing the backgrounds of the computer screens and modifying the computer’s system preferences. I had heated debates with my friend at the time about how PCs were better than Macs. (I now own a Mac.) Looking back the further you go the easier is to see how. It. all. Made. sense. 

I majored in Political Science and attended a coding bootcamp that folded midway through my cohort. I interviewed for what I envisioned to be my dream job at the time - Developer Envangelist at Square, traveling the world, writing blog posts and tutorials. But then failed the interview miserably. After a dismal performance the interviewer, very ready to get off the phone asked if I had any final questions. “What do you wear to the office?”. Shorts and flip flops apparently. The role is still open. I’ve made a GitHub repo of all the resources I can find to aceing coding interviews. I like collecting the information but not wasting my time on the exercises. It begs the question, well how are you going to advance in your career as a Software Engineer if you’re not inverting binary trees and can’t smash out Leetcode problems. How will anyone hire you? What I’m doing to become a better programmer is working hard at my job to make the product better and ship features that work. I’m building a side project that people use and it’s my goal this year to get people to pay money for it. Here are some new years resolutions I set in Q1 that I’ve completed. I didn’t get tooo crazy about SMART goals and objectives and key results and everything but I feel pretty good about marking them as checked off the list.

### Completed goals 2020

These are New Years resolutions I set in January that I’ve completed:

**Get my cholesterol down to a healthy limit**. This year I’ve been eating less meat and trying to be healthier. I trained for a marathon and was going to the gym lifting weights. I fasted for two days and took a blood test where my cholesterol dropped tremendously. I’ve also been taking cholesterol lowering meds. This goal came about because I’d tested really high before and I have a family and I want to be healthy and be here for them and to feel good and enjoy life. This is something I’ll definitely have to keep an eye on and keep testing for but since I took the test and the numbers were low checking this off the list as a goal I’ve hit.

**Get personal site up and running**. I blogged about migrating my site from Hexo to Hugo. Gotta love the names of these static site generators. I also moved from hosting on Github Pages to Netlify. The result of this transition is I feel more comfortable deploying to my site and like the look and feel more. It’s easier to make changes. People, myself included going back and forth about their blog hosting. I think the most important thing is to write consistently and feel comfortable editing and making updates. I’m a software engineer so I don’t think I’ll ever really stop playing with this site but for now it’s working and after this transition marking completed as the site being “up and running”. Maybe next year or something I’ll migrate to a PHP site like Statamic but for now not tripping. Another checkmark on the list.

**Redesign Employbl homepage**. I worked with my wife to do a redesign of Employbl. New fonts, new logo, new imagery, new colors. The site is live and I’m happy with it. The logo on there before was something free from the internet. It wasn’t too bad but I like it now knowing that it’s my own and that I’ve thought deeply about color choices etc instead of spur of the moment Bootstrap CSS classes cause why not. Improving the design and usability and stickiness of the website is a never ending project but checking off the homepage redesign as complete.

**Easy candidate approval process on Employbl (and fix onboarding bugs)**. This I’ve also blogged about. Previously it was pretty time intensive to approve and add candidates to Employbl. Last year candidates were still filling out a Google Form on the Laravel app to join the candidate network and upload their resume. The login with LinkedIn was buggy and didn’t work at all for some candidates. I’ve migrated it over so the social login is reliable and uses Auth0 and our own MySQL database to verify users. Approving users is as straightforward as running an artisan command on our production server. There are definitely optimizations here like allowing candidates to upload resume + provide more information when they first apply instead of after login. We could also make an easy approval button so I can do it from my phone instead of SSH’ing into the server but for now it’s set up so it works and doesn’t take me much time or manual effort at all to approve candidates and welcome them into the Employbl network.

**Companies on Employbl can have multiple locations**. This one I didn’t do anything on but I began working with a former co-worker on Employbl and he built all the functionality. It has been a breath of fresh air bringing a co-founder onto the project. I’ve stayed more consistent while working on the project as we generally have check ins once a week. 


### Goals in progress

**Host 12 meetups**. I run the full stack developer meetup in Oakland. Last year I did a few events that went great and it’s my goal to do one event a month. We had events in January and February. I had the space and speaker booked for March but then the pandemic happened and I had to cancel. I’ll be launching a remote event happening in a couple weeks covering AWS Lambda and the Serverless Framework. Excited to keep this going, even if it’s remote.

**10x traffic to Employbl to 20k users per month**. There have been a few traffic spikes but overall traffic has stayed consistent trending slowly upwards. Going to need to focus on this one to make it happen. Traffic over the past 30 days currently sits at 3,400 users. Tracking the users in Google Analytics not candidate or recruiter signups.

**Display featured candidate for recruiters on Employbl**. This is a goal I set as kind of an intermediary set. Ultimately I’d like to have full text search on the PDF resumes that candidates upload to Employbl. I’ve never really played with Elastic Search so figure it might be a pretty big project to set it up, build the UI and search text in PDF files. This step is to display “featured candidates” to recruiters that I pick and maintain based on the job listings recruiters using the platform are hiring for. It’s not perfect but it would necessitate building a login portal and dashboard for recruiters which definitely needs to happen.

**Elastic Search for employers to search all candidates**. This is the second step of the process building badass search functionality so recruiters can find candidates. I’m thinking I’m also going to need to change the information candidates provide so we get their resumes and some more structured information about their experience and what they’re looking for. This will be some of the final changes to candidate onboarding/data collection before shifting to focus on capabilities for recruiters. 

**Run a marathon**. I was doing pretty good on this one until the Oakland Marathon got rescheduled to August. As soon as I heard it was canceled I pretty much stopped doing everything. I still haven’t run a marathon and it’s going to be brutal but still something I want to check off my bucket list this year. Even if I can’t run in a group and it’s just me running 26.2 miles around the town that counts. Still need to get this done.

**Run a sub 20 minute 5k**. In addition to running a marathon I’d like to run a fast 3.1 miles. This training is a little different with more speed and strength work where I feel like marathon training is long, drive you crazy mileage. Both worthy goals but I need to get back on track and start logging some miles to makes these ones happen.


## Personal action items for Q2

I’d definitely like to make progress on my health and fitness this quarter. Last year I did some pretty significant Freeletics bodyweight workouts that had me feeling good. I think I can do these in conjunction with running to get in good shape and prepared for these marathon and 5k races. I have a training schedule on Airtable where I track my workouts and progress (or lack thereof…). For this quarter I’d like to start consistently logging **20-25 miles per week** and **3-4 Freeletics workouts per week**. This’ll be a pretty significant uptick as in the past week I ran about ten miles with zero Freeletics workouts lol.

Another big focus for me has been excelling in my day job. I’m on track to move into a sprint engineering role with Laravel, Javascript, PHP and AWS. There’s a lot of work there. My 9-5 work goals and parenting goals and financial goals and everything I’m tracking separately but definitely a big focus for me going on this quarter too.

Okay so Q1 2020. Good times. Really I wanted to write this blog post because I need to get in shape and I figured that writing something down stating my commitments would a) clarify my thinking and get me going since I’d have to write it down b) hold me accountable or give me something to look down on and c) drive SEO and get me writing, make me more comfortable writing and publishing things on the internet and d) I actually did knock out a bunch of my New Years Resolutions in Q1 and have some big ones to go so I want to share my journey and be a part of something bigger and motivate myself and others and koombyeah and all that jazz :)


