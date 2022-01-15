---
title: "Deploy Spring boot Jar Docker to Heroku using two different ways"
date: 2022-01-15
description: "Deploy Spring boot Jar Docker to Heroku"
categories:
  - "Development"
tags: 
    - "Development"
    - "Spring Boot"
    - "Java"
    - "Docker"
    - "Heroku"
#math: false
lead: "A short tuts on deploying spring boot jar docker to heroku"
comments: true
mathjax: true
toc: false
authorbox: true
sidebar: "right" 
#menu: main
---
{{< figure src="/images/heroku-image.png" caption=" " >}}
Imagine you and your buddy are developing a Spring boot Rest application using Java 11. To make it run consistently in yours and your buddy’s machine you decide to setup Sprint boot jar in Docker.

Now the docker setup is done, you want to release it to world and announce it in Product hunt.

Out of many cloud provider like AWS, Google cloud; Heroku is the most beginner friendly, whilst offering advanced features which can be scaled for a production level application.

In this article I’m going to explain the different steps involved in deploying a Spring jar docker to Heroku.

## Heroku Deployment

Heroku is a popular Platform-as-a-Service(PaaS) cloud provider. It makes it easy to deploy our application to the cloud.

Before that we need to create an application in Heroku. 

### Application setup in Heroku

1. Go to [https://dashboard.heroku.com/](https://dashboard.heroku.com/)
2. Create a new login
3. Click create new App on the right top corner,

{{< figure src="/images/heroku-new-app.png" caption=" " >}}

Then click `Create new app`

You’ll be prompted to provide new name. Provide new name and select an appropriate region. This region specifies in which region your application will be saved in.

1. This creates an app in Heroku. Now we need to push our docker to this app. 

Before we dive into that lets first checkout in which way we can to deploy

 dockers to heroku.

### There are two Ways to deploy Docker to Heroku

### Method 1: Deploy pre-build docker images

In this method if we pre-build docker images, and then release it to Heroku container.

1. Build docker image

```yaml
docker build -t testApp .
```

2. Once docker image is build check if its present using below command

```yaml
docker images
```

3. Login to container Registry

```yaml
heroku container:login
```

4. Create your Heroku application. (This can be skipped if you have created Heroku app via Heroku dashboard)

```yaml
heroku create <app-name>
```

5. Build the image and push to container registry

```yaml
heroku container:push web
```

6. Release the image to Heroku App

```yaml
heroku container:release web
```

7. Now open app in browser

```yaml
heroku open
```

### Method 2: Build Docker images in Heroku

In this method Heroku will build our docker images and deploy it to specified container

1. First, we need to create a `heroku.yml` file where we define Dockerfile details

```yaml
build:
  docker:
    web: Dockerfile
```

2. Then, create the docker file

```yaml
#Create Image
FROM openjdk:11-jdk
MAINTAINER test@gmail.com
COPY target/test.jar test-services.jar
CMD [ "sh", "-c", "java -Djava.security.egd=file:/dev/./urandom -jar /test.jar" ]
```

Here we copy our jar named as `test-services.jar` to target as `test.jar`

And the application is started at line `CMD` where application jar i.e `test.jar` is specified.

3. Now login to Heroku using below code 

```yaml
heroku login
```

4. Set the new Heroku app to use container using below command

```yaml
heroku stack:set container --app your-Heroku-app-name-here
```

5. Now if open the Heroku dashboard you will see the application is configured to use container. 
{{< figure src="/images/heroku-container.png" caption=" " >}}

6. Now commit to git and push code to Heroku branch

```yaml
$ git add .
$ git commit -m "docker setup"
$ git push heroku master
```

That’s it, your application should be deployed to Heroku.

But wait! there more....

## Issues faced

These steps should be enough to deploy to Heroku,But these are some common errors which you might encounter. 

Some of them are listed below,

### Java 11 issue

By default Heroku runs in Java 8. To make it run in OpekJDK 11, add below line in `[system.properties](http://system.properties)` file in the root of the folder

```yaml
# Heroku configuration file
# see https://devcenter.heroku.com/articles/java-support#specifying-a-java-version
java.runtime.version=11
```

### Preventing Error R14 (Memory quota exceeded)

After all this, you might still face errors in Heroku. If you check the Heroku logs( `heroku logs t`, you’ll find the R14 error. 

```
$ heroku logs --tail --app your-Heroku-app-name-here
2019-07-24T02:58:48.253177+00:00 heroku[web.1]: Process running mem=836M(163.4%)
2019-07-24T02:58:48.253243+00:00 heroku[web.1]: Error R14 (Memory quota exceeded)
2019-07-24T02:58:55.236933+00:00 heroku[web.1]: State changed from starting to crashed
2019-07-24T02:58:55.111947+00:00 heroku[web.1]: Stopping process with SIGKILL
2019-07-24T02:58:55.217642+00:00 heroku[web.1]: Process exited with status 137
```

To resolve this we need explicitly set memory to the JVM, add below to CMD line

```yaml
CMD [ "sh", "-c", "java -Xmx300m -Xss512k -XX:CICompilerCount=2 -Dfile.encoding=UTF-8 -XX:+UseContainerSupport -Djava.security.egd=file:/dev/./urandom -jar /test-services.jar" ]
```

### Preventing Error R10 (Boot timeout)

Another error which we encounter is R10 or Boot timeout error. This may look like below in Heroku logs,

```yaml
2019-07-24T02:58:55.236933+00:00 heroku[web.1]: State changed from starting to crashed
2019-07-24T02:58:55.111947+00:00 heroku[web.1]: Error R10 (Boot timeout) -> Web process failed to bind to $PORT within 60 seconds of launch
2019-07-24T02:58:55.111947+00:00 heroku[web.1]: Stopping process with SIGKILL
2019-07-24T02:58:55.217642+00:00 heroku[web.1]: Process exited with status 137
```

To fix this we need to specify the port like below,

```yaml
CMD [ "sh", "-c", "java -Dserver.port=$PORT -Xmx300m -Xss512k -XX:CICompilerCount=2 -Dfile.encoding=UTF-8 -XX:+UseContainerSupport -Djava.security.egd=file:/dev/./urandom -jar /oota-services.jar" ]
```

### The complete docker file is as below,

```yaml
#Create Image
FROM openjdk:11-jdk
COPY target/test.jar test-services.jar
CMD [ "sh", "-c", "java -Dserver.port=$PORT -Xmx300m -Xss512k -XX:CICompilerCount=2 -Dfile.encoding=UTF-8 -XX:+UseContainerSupport -Djava.security.egd=file:/dev/./urandom -jar /test.jar" ]
```

## Conclusion

So that’s how you go about deploying spring boot Jar in Heroku. Hope you find this article useful.

If your interested in deploying a Spring boot War in docker, the article for the same is linked below,
[Docker Article](https://chiragsp.dev/post/spring-boot-war-docker/)
