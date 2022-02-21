---
title: "How I built my first Saas product"
date: 2022-02-19
description: "Journey of my first Saas product - SQL Grammar"
categories:
  - "Blog"
tags: 
    - "Blog"
#math: false
lead: "Building my first Saas Product SQL Grammar under a month"
comments: true
mathjax: false
toc: false
authorbox: true
sidebar: "right"
menu: main
---
So its the start of February 2022. Already a month of 2022 has passed. Guess I could stop wishing people happy new year from now on 😏.

At the start of this year I had taken upon a challenge of building something and publishing on Product Hunt 🚀.

You know, one can’t really comprehend how tough it is to build and actually release something to the world; until you actually do it. I have new found respect for creators everywhere now.

So coming to January 1st 2022, I had just decided I going to build an SQL linter to check my SQL statements.

This is how `SQL Grammar` was born.

So as someone who has no idea what he/she is getting into, I went ahead and bought the domain `sqlgrammar.com`

I was surprised that I was actually able buy this domain under 10 dollars. This was a good sign I hoped.

So time had come to decide on how to go about building this and how to fit this with my schedule.

Some background on myself for context, I work as a full time Senior Full-Stack developer in a large enterprise Product company, mainly working on Java and Angular.

I knew too well, if this project had to be completed on time, I had to do it in the mornings. I decided to wake up at 5:30 am from now. I decided on below schedule

1. Work on `sqlgrammar.com` from 5:30am to 7am
2. Workout from 7am to 8am
3. Breakfast from 8am to 9am
4. Work on additional side project which I am doing with my cousins for a few months now(will explain later). 9am to 10am
5. Blog. 10am to 10:30am
6. Work on my day job. 10:30am to 7:30am
7. Be in bed by 11pm.

This schedule I thought is the best I could manage. With this I could focus on side projects and work separately, and also get some time at night to relax and unwind.

Something amazing happened on January 1st. I actually got up at 5:30 am. And surprisingly, I kept getting up at 5:30 am for the entire month. (The secret I found is setting a schedule in google calendar for the next day 🤫).

During the first week, I decided to focus on building the front-end for `SQLGrammar`

Front-end has always been my weak spot. Even though I started as a front-end developer 8 years back(this is was when jQuery was actually used 🥲), I had completely switched being Java back-end developer in the last 6 years. Its only in the past year that I had gotten the opportunity to build something in Angular.

But still my HTML/CSS game was weak. So I decided to leverage `Ng-Bootstrap` for this project.

So I set out to draw some sketches on how the product will be.

Some of the first sketches I drew are shown below,

{{< figure src="/images/initial-blueprint-1.jpg" caption=" " >}}
{{< figure src="/images/initial-blueprint-2.jpg" caption=" " >}}

The initial version of the application would be very bare bones. The idea was that the app will call SQL multiple times and will display the subsequent syntax error instead of actually writing a parser. 

So my vision for this product was that, an user can check a SQL statement for 10-15 times for free and get feedback suggesting if SQL is correct or not. After that ask user to pay for subscription for unlimited SQL checks.

So time had come to decide the Tech Stack.

I was good in Java Spring boot but for this project my thought process was that it required a very simple back-end, overhead of Java and Spring boot was a bit too much. So I decided to go with `nodeJS` along with `expressJS.`

As for the front-end, and this being my weakspot, I decided to stick with what I know and have experience on. Since for the past one year I had been working on an Angular project in my day job, I decided to stick with Angular. 

And now coming to databases, since I wanted to support multiple databases, and since I did’nt have enough credits the buy in AWS 🙄, I decided as a start I’ll stick with `MySQL`.

So to recap, this is the tech-stack I decided upon.

Back-end : Nodejs/ExpressJs

Front-end: Angular/Ng-Bootstrap

DB: MySQL

Only thing remaining was hosting. This I thought to worry about later. Big mistake 😪. Will come back to this at the end of article.

So finally I dove down to coding the back-end part first. Coding with expressJs is so much fun. Its so easy to setup and write API REST calls with it.

Wrote up a method to connect to my local MySQL and API to get the SQL data from front-end.

Plan was to directly execute the SQL statement and to ignore common errors like ‘table doesn’t exist’ errors. I know its very risky and crazy plan(This is security nightmare btw 😶). But I wanted to start somewhere.

It was very rudimentary but was good for the first version.

Now I  opened up `Postman` and tried calling the API, and surprisingly was successful in making a successfull call on first try (never a good sign 🤔)

So after making it work in my local, the next step was to setup a Cloud Database.

I being very cheap started searching for a free DB hosting platform. I had previously setup side projects on Heroku and I knew it offered only PostGres for free, and that too only 20 connections. This was far too less for SQLGrammar.

Finally after researching for a good two hours, I decided to try out AWS. I had no prior knowledge on AWS, so for next couple of days I spent learning and researching on what is a S3 bucket, what is RDS etc.
I struggled for quite number of days on AWS and but was finally was able to setup AWS and RDS. And finally was able to deploy my back-end application on it.

Now coming to building out the front-end, as mentioned earlier I chosen to build in Angular. I being not much of a designer, had decided to use pre-build components. So I chose to use Bootstrap. I knew how to use Bootstrap before, but getting it installed and working in Angular was a big challenge. 

I struggled for a couple of days(more actually 🤫) but was ultimately successful in getting it up and running. This showed how weak my fundamentals are with regards to Angular and also gave me an opportunity to improve my Angular skills.

For the front-end, I kept it simple. I setup a textbox and drop-down along with a header navbar and footer.

Initially when I started this project, I assumed building a front-end would be tough , but I actually enjoyed designing the front-end. Even though there were challenges, it was fun.

Finally time had come to deploy the front-end application. After researching a bit I was able to deploy it to Heroku. All was well and I thought I’ll be done by January 20th.  Only thing left was connecting to it my external domain, and adding better messages. 

Now came my first major road block. Heroku deploys to a https site and my backend was in http. So when my front-end code tries calling my backend code, I was getting a CORS error. 

After a lot(i mean a lot 😓) of research , trial and error, I was able to figure out the root cause.
So apparently Heroku prevents calls to a Http backed from a Https frontend. 

This was a major blocker and I spent couple of days trying to resolve it, but was getting nowhere.
Then just as a distraction I started searching if there are any SQL linter libraries available. That’s when I came across a great python library called SQLFluff([https://www.sqlfluff.com](https://www.sqlfluff.com/)).

This is a basically a SQL linter and it’s free to use. This was great.  With this I didn’t even need a DB. I just had to build a python back-end and use SQL fluff library in it. So I quickly went over on how to develop python REST application and came across `Flask RESTful`. After going over few videos and documentation , I was able to quickly build up a Python REST back-end. Did a quick test in local and was successful.

This time around, I didn’t even need a DB in back-end, so I decided to go the free route and deploy the python flask app in Heroku. After connecting this to the front-end application already existing in Heroku, all was working , no more CORS error 🕺🏻.

Now came the part about monetizing the application. I felt I didn’t really provide any extra value as, parsing was done by SQLFluff and the front-end needed a lot of UX improvements. So I decided to keep it free to use. 
But I wanted to atleast to limit how many language an anonymous user could use. So I decided for languages other than MySQL and PostGres, I’ll prompt the user to login. Decided to use Google login for this purpose. After researching a bit, I decided to use `Firebase` for authentication.

During this I released it would easier to just deploy the entire front-end application in `Firebase` as it provides many features out of the box and there is a free tier as well. I spent the next 2-3 days on this deployment, and was able to complete this.

Now we come to last week of January. With the Product Hunt release looming, I focused on fixing all the bugs and improving the UI a bit. Also, I wanted to write something for my Product Hunt listing page.

I had not sold anything in my life before and had no idea how to do a Sales pitch. So started finding out some Saas founders I follow on twitter and what their early SaaS products looked like and how their landing page looked. 

After going through some of them, I decided to record a GIF on how to use `SQlGrammar`

I was able to put together something like this,
{{< figure src="/images/MVP-1.png" caption=" " >}}
{{< figure src="/images/MVP-2.png" caption=" " >}}

Finally it was January 30th, the final day 😎.
Started setting my product hunt page, added the screenshot , GIF and added a brief description.

At this moment I thought to mysql, should I even launch this product? I cowardly tweeted out to my (6 followers 🥲) about SQL grammar. I was thinking this qualifies as a launch and the task was done. 

But something happened the next day. I decided I had to launch on Product Hunt. I figured I won’t care if I don’t get any upvotes, all that mattered was showing SQLGrammar to the world.

I scheduled a launch for next day but due to mix up in time difference(my mistake 😐) I scheduled the launch to Feb 02nd on Product Hunt and waited.....

In Product hunt, products get listed at PST 12:00am. That works around 1:30pm in my time zone. So at 1:30pm on Feb 03rd 2022 with baited breath I opened Product Hunt Page. And Voila! SQLGrammar was listed. I was super happy and proud to see SQLGrammar in the listing. I was just hoping atleast some people would visit 🙄.

Then the comments started coming, and all of them very positive and very encouraging and `SQLGrammar` ended up at rank 21 by end of day. I was over the moon. This was better than I expected.

[https://www.producthunt.com/posts/sql-grammar](https://www.producthunt.com/posts/sql-grammar)
{{< figure src="/images/Product-hunt.png" caption=" " >}}

I was ecstatic. I showed my Mom and sister, called my friends and told them about SQLGrammar. This being my first attempt, it was very encouraging. I would have never completed and launched if I hadn’t given myself a challenge at the beginning of the year.

So for February I decided to do something simpler as I will away for 10 days vacation and won’t be able to code.

I decided that I will write a notion template to prepare for Java Interviews and try selling it on Gum road.

Aim for this project to make a dollar from this template. And as for March I am still unsure about what to do. Should I work improving SQL grammar or start working on a new SaaS idea. 

It has been a very encouraging start of the year, and I hope I end up completing my goal for this year, i.e to earn something outside my day job.

Wish me luck 🤗

Do visit SQL Grammar and leave a comment

www.sqlgrammar.com
