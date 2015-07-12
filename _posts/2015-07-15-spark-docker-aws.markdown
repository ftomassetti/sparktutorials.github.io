---
layout: post
title:  "Spark, Docker and AWS"
author: <a href="http://tomassetti.me" target="_blank">Federico Tomassetti</a>
date:   2015-07-15 10:00:00
summary: >
  Docker is gaining popularity as a tool to make deployment testable and repeatable. In this tutorial we will see how to define a docker container, how to use it during development and testing and how to use to deploy to AWS.
---

Docker is gaining popularity as a tool to make deployment testable and repeatable. In this tutorial we will see how to define a docker container, how to use it during development and testing and how to use to deploy to AWS.

Note that Docker runs natively on Linux. You can still use it with Windows or the Mac, you just need to use a sort of wrapper named *boot2docker*. Basically you need to give a couple of commands more to install and start boot2docker and then use Docker as you were on Linux.

##Overview of the application

We will base this tutorial on the same blog application that we have developed in previous posts: feel free to stop and take a look at the other tutorials if you are new to Spark!

Our application will run on two docker containers:

* the first container will run a postgres database
* the second container will run the spark application

As for previous posts the code is available on [GitHub](https://github.com/sparktutorials/BlogService_SparkExample).

##The database

In the first container we will run a postrgres server, version 9.4. This container is based on ubuntu but feel free to change it. We need also to set permissions to give access to the server from any IPs. As you can see one of the benefits of Docker is that it forces us to precisely define every aspect of the environment: instead of instructions like "add this line to the pg_hba.conf file in directory /etc/postgresql/9.4/main/" you have a precise recipe to build the environement. Isn't this great? 

So to define the container create a file named _Dockerfile_ and put it a directory named *db_container*.

<pre><code class="language-bash">
FROM ubuntu:latest
MAINTAINER Federico Tomassetti

RUN apt-get install -y wget
RUN echo "deb http://apt.postgresql.org/pub/repos/apt/ trusty-pgdg main" >> /etc/apt/sources.list.d/pgdg.list

RUN wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

RUN apt-get update
RUN apt-get install -y python-software-properties software-properties-common
RUN apt-get -y install postgresql-9.4 vim
RUN echo "local   all             all trust" >> /etc/postgresql/9.4/main/pg_hba.conf
RUN echo "host    all             all             0.0.0.0/0            trust" >> /etc/postgresql/9.4/main/pg_hba.conf

RUN echo "listen_addresses='*'" >> /etc/postgresql/9.4/main/postgresql.conf

ADD ./setup.sql /db/setup.sql
ADD ./schema.sql /db/schema.sql

# We start the server and setup the database
RUN /etc/init.d/postgresql start && sudo -u postgres psql < /db/setup.sql && /etc/init.d/postgresql stop
# We start the server and define the tables
RUN /etc/init.d/postgresql start && sudo -u postgres psql -d blog < /db/schema.sql && /etc/init.d/postgresql stop

EXPOSE 5432

USER postgres

# Each time we run the container the server is started
CMD ["/usr/lib/postgresql/9.4/bin/postgres", "-D", "/var/lib/postgresql/9.4/main", "-c", "config_file=/etc/postgresql/9.4/main/postgresql.conf"]
</code></pre>

We have seen how to define the database container. Now we need to build the container and run it. 

###Build the database container

I guess you figured out you need to install [Docker](https://www.docker.com/) by now. If you have not yet install it this is the right time. Don't worry, I can wait.

To generate a container from the Dockerfile you can run this command:

<pre><code class="language-bash">
docker build -t blog_db_container db_container 
</code></pre>

It takes the Dockerfile in the directory *db_container* and build a container tagging it as *blog_db_container*

Now if you run:

<pre><code class="language-bash">
docker images
</code></pre>

You could see an entry named *blog_db_container*.

###Run the database container

Let's start the container now:

<pre><code class="language-bash">
# run the container mapping the postgres server port (5432) to the same port on the
# host machine (5432)
docker run blog_db_container
# run the container mapping the postgres server port (5432) to port 6000 on the
# host machine
docker run -p 6000:5432 blog_db_container
</code></pre>

Wonderful. Now, how can we see if it works? Let's try to connect to the postgres server. Note that you need to install the postgres client on your development machine. So far we installed the postgres server in the container but we did not install anything on the real machine. To connect to the server run this command: 

<pre><code class="language-bash">
# We suppose you expose the docker database container on port 6000
psql -h localhost -p 6000 -U blog_owner -d blog
</code></pre>

Note that when connecting we specified the username and the name of the database. These values are defined in the *setup.sql* script.

##The Spark application

##Deploying on AWS
bar

##Conclusions
I am using docker more and more mainly because it offers me the possibility to define precisely an environment and test against it: for example our docker container contains a specific version of the postgres server (9.4.1) and our applications is tested always with this specific settings. Before using docker I had to manually install postgres and verify my version matched the one used in deployment. Is very easy to overlook such details and the result could be a potential surprise the moment you deploy your application. In addition to that using Docker make easy to switch to other environements: you change a line in your Dockerfile and you can test your application with a different version of Postgres. Finally you have the benefit of having the configuration of your system (which is defined by the Dockerfile) under version control and I find this awesome.

{% include authorTomassetti.html %}