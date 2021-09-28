---
title: "Opentext TGO portal TGFIA Jar"
date: 2019-12-01
slug: "portfolio-tgfia-jar"
description: "This a maodule part of larger TGO portal"
keywords: ["Java" , "Tomcat" , "SOAP API" , "Jersey 2" , "Struts 2"]
draft: true
tags: ["Java" , "Tomcat" , "SOAP API" , "Jersey 2" , "Struts 2"]
math: false
toc: false
---

This product was part of a larger TGO application in Opentext - which is a large enterprise application that is used to manager mailboxes by different companies
TGFIA is basically a directory of different mailboxes that different modules of the component use to get mailbox data.

This module was initially in java 6 and was consuming SOAP API calls to get data from other modules of larger TGO application.
My work involved migrating TGFIA from Java 6 to Java 8. Also to move the client SOAP API calls to REST calls. In addition to this , had to move it from Tomcat 6 to Tomcat 8.

As this was old legacy product(updated 6 years prior) , there was no local setup or anyone to lean on get additional info
This involved learning about struts 2, tomcat 6/8 and also client side SOAP API and client side REST API