---
layout: post
title: Quickstart your SpringBoot with Spring Initializr
description: "A quick overview of the excellent Spring Initializr CLI"
modified: 2016-05-23
tags: [sample post]
image:
  feature: abstract-3.jpg
  credit: ddewaele
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
---

#Introduction

Spring offers a [Spring Initializr](https://start.spring.io/) to bootstrap your application quickly
The most simple way to use it is to do this :

{% highlight shell %}
curl https://start.spring.io/starter.zip -o demo.zip
{% endhighlight %}

It will download a `demo.zip` file to your filesystem that you can extract:

{% highlight shell %}
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 50954  100 50954    0     0  38634      0  0:00:01  0:00:01 --:--:-- 38660

ls -ltr
total 104
-rw-r--r--  1 ddewaele  staff  50954 May 21 00:54 demo.zip
{% endhighlight %}

The zip file contains a complete spring boot application.

{% highlight shell %}
unzip demo.zip 
Archive:  demo.zip
  inflating: mvnw
   creating: .mvn/
   creating: .mvn/wrapper/
   creating: src/
   creating: src/main/
   creating: src/main/java/
   creating: src/main/java/com/
   creating: src/main/java/com/example/
   creating: src/main/resources/
   creating: src/test/
   creating: src/test/java/
   creating: src/test/java/com/
   creating: src/test/java/com/example/
  inflating: .mvn/wrapper/maven-wrapper.jar
  inflating: .mvn/wrapper/maven-wrapper.properties
  inflating: mvnw.cmd
  inflating: pom.xml
  inflating: src/main/java/com/example/DemoApplication.java
  inflating: src/main/resources/application.properties
  inflating: src/test/java/com/example/DemoApplicationTests.java
{% endhighlight %}

You can customize the cURL command by providing some additional parameters (groupId,artifactId,....)

{% highlight shell %}
curl https://start.spring.io/starter.tgz -d dependencies=web,actuator -d groupId=com.ecs -d artifactId=hawkbit.client -d language=java -d type=maven-project -d baseDir=hawkbit.client | tar -xzvf -
{% endhighlight %}


# References

[Spring initializr Github](https://github.com/spring-io/initializr/)
[Spring Starter](https://start.spring.io/)
