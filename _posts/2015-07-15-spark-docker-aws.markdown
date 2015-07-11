---
layout: post
title:  "Spark, Docker and AWS"
author: <a href="http://tomassetti.me" target="_blank">Federico Tomassetti</a>
date:   2015-07-15 10:00:00
summary: >
  Docker is gaining popularity as a tool to make deployment testable and repeatable. In this tutorial we see how to develop an application using two containers: one for the database and one for the Spark application itself.
---

##Overview of the application

We will continue using the blog application we have developed in previous posts. Our application will run on two docker container:

* the first container will run a postgres database
* the second container will run the spark application

As for previous posts the code is available on [GitHub](https://github.com/sparktutorials/BlogService_SparkExample).

##The database

##The Spark application

##Deploying on AWS
bar

##Conclusions
I am using docker more and more mainly because it offers me the possibility to define precisely an environment and test against it: for example our docker container contains a specific version of the postgres server (9.4.1) and our applications is tested always with this specific settings. Before using docker I had to manually install postgres and verify my version matched the one used in deployment. Is very easy to overlook such details and the result could be a potential surprise the moment you deploy your application. In addition to that using Docker make easy to switch to other environements: you change a line in your Dockerfile and you can test your application with a different version of Postgres. Finally you have the benefit of having the configuration of your system (which is defined by the Dockerfile) under version control and I find this awesome.

{% include authorTomassetti.html %}