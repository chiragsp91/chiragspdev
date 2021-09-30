---
title: "Opentext TGO - TGFIA Services"
date: 2020-06-02
slug: "portfolio-tgfia-services"
description: "This a maodule part of larger TGO portal"
keywords: ["Java" , "Tomcat" , "Spring Boot" , "Swagger UI" , "Oracle 11g", "Ansible" , "New Relic"]
draft: true
tags: ["Java" , "Tomcat" , "Spring Boot" , "Swagger UI" , "Oracle 11g", "Ansible" , "New Relic"]
math: false
toc: false
---

This product was part of a larger TGO application - which is a large enterprise application that is used to manager mailboxes by different companies
TGFIA is basically a directory of different mailboxes that different modules of the component use to get mailbox data.
This project involved converting TGFIA from a standalone jar to a REST Service.

TGFIA was a standalone jar, that was used by each application to get data from TGFIA DB and also to insert data to TGFIA. This various issues in updating TGFIA jar. As any kinda of update or fix to TGFIA jar involved, asking all stakeholders using the tgfia jar to get the required updated and test, or them not getting required update and ending up in a scenario having many versions of TGFIA at the same.

So it was decided to move TGFIA from standalone jar to a REST service , which the variosu clients would call to interact with TGFIA data.