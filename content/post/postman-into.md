---
title: "How to use POSTMAN Application"
date: 2020-07-25
description: "A quick introduction on how to use postman"
categories:
  - "Development"
tags: 
    - "Development"
    - "Postman"
lead: "A Postman setup guide"
comments: true
mathjax: true
toc: false
#menu: main
---

REST APIs being gold standard in the industry these days , every application uses REST APIs. 

So how to do you test it. You can use a browser to call the API and use an extension like JSONViewer to see the returned JSON response. If the returned data in in XML, we need another extension. So quickly realize you need complete set of tools to test REST APIs.

This is were postman comes in. Postman provides a comprehensive set of tools to do manual testing of REST APIs.

### Step 1 : Download POSTMAN application from below link

[Download Postman App](https://www.postman.com/downloads/)

### Step 2: Once you download postman, you should have something like as shown below
{{< figure src="/images/postman-1.png" caption=" " >}}


### Step 3:  Let us setup an example GET request.

To test postman we will using sample APIs provided by POSTMAN. We can get the sample APIs here,

[Postman Echo](https://docs.postman-echo.com/?version=latest)

We will use an GET request as below
{{< figure src="/images/postman-2.png" caption=" " >}}


### Step 4: Now lets do a POST request

We will use a POST request as below
{{< figure src="/images/postman-3.png" caption=" " >}}


Similarly if you intent to PUT and POST , can change the type of request here ,
{{< figure src="/images/postman-4.png" caption=" " >}}


### Bonus Step: How to send JSON/XML or file as input

You can set the JSON/XML body to send as shown below
{{< figure src="/images/postman-5.png" caption=" " >}}

The way to send file as a input in POSTMAN is as shown below
{{< figure src="/images/postman-6.png" caption=" " >}}



