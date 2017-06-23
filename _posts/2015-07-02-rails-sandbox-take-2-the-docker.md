---
layout: post
title:  "Rails Sandbox Take 2: The Docker"
cover: https://static.pexels.com/photos/4463/man-person-blue-color.jpg
tags:
  - en
---

Few posts ago i wrote about [rails sandbox](/posts/rails-sandbox) and isolated [development environment](/posts/your-development-environment) using Vagrant and Chef Solo. But there's more options available. One of them is gets more and more traction and gains popularity. It's the Docker.

<!-- more -->

## The Docker

Docker in it's concept is similar to Vagrant but yet different. It also wrap your software in separated environment called container as Vagrant does with Virtual Machine. But the difference is that containers doesn't posses it's own OS but it's share kernel with other containers. They run in isolated userspace on the host operating system.

## The Tutorial

If you haven't worked with Docker before you should go through [Docker tutorial](http://docs.docker.com/mac/started/). It explains all basic concepts of working with Docker.

## The Configuration

Container configuration in concept is similar to Vagrant it sits in single file `Dockerfile`. So let's say we want to prepare container to work with Ruby on Rails.

Not digging into further details let's describe what we'll need to run rails application on Ubuntu machine

* basic development tools (build-essential package)
* javascript runtime (nodejs package)
* nokogiri dependencies (libxml2-dev libxslt1-dev)

The we need add our application to container, install all necessary gems and we're ready to go.

Let's roll up sleeves define the `Dockerfile`

{% highlight dockerfile %}
FROM ruby:2.2.2

RUN apt-get update -qq && apt-get install -y build-essential

# for js
RUN apt-get install -y nodejs

# for nokogiri
RUN apt-get install -y libxml2-dev libxslt1-dev

# create folder for application
ENV APP_HOME /app/rails_application
RUN mkdir -p $APP_HOME
WORKDIR $APP_HOME

# add Gemfile and Gemfile.lock
ADD Gemfile* $APP_HOME/
RUN bundle install

# copy over application
ADD . $APP_HOME
{% endhighlight %}

Now we need to build this image and it'll be ready to use. `docker build -t image_name .`

To run rails application we need to issue command `docker run image_name rails server`.

## The But

But there are some problems. Code is embedded into image, we don't know what ip has container and we cannot go through firewall at standard rails port 3000.

## The Work

Docker client takes numerous of parameters but right now only few of them are needed.

* `-v /host/path:/container/path` will mount path from host system into container
* `-p host_port:container_port` will expose ports to host

Now we need to run `docker run -p 3000:3000 -v ~/code/docker/rails-app:/app/rails_application image_name rails s -b 0.0.0.0`

This will start rails application as you usually do but in docker container.

YAY! We could quote "That's all folks!" but what if ...

## The Orchestration

... what if we want to be more complex, have separate container for different parts of application. Let's say we need separate containers for PostgreSQL and Redis. How to handle this?

I say we could use `docker-compose` tool. All what it needs is another file that will hold whole configuration, `docker-compose.yml`. But we need adjust out `Dockerfile` first and install packages for postgres client.

{% highlight dockerfile %}
FROM ruby:2.2.2

RUN apt-get update -qq && apt-get install -y build-essential

# for js
RUN apt-get install -y nodejs

# for nokogiri
RUN apt-get install -y libxml2-dev libxslt1-dev

# for postgres
RUN apt-get install -y libpq-dev postgresql-client

# create folder for application
ENV APP_HOME /app/rails_application
RUN mkdir -p $APP_HOME
WORKDIR $APP_HOME

# add Gemfile and Gemfile.lock
ADD Gemfile* $APP_HOME/
RUN bundle install

# nifty trick to clean pid leftovers
RUN test -f $APP_HOME/tmp/pids/server.pid && rf $APP_HOME/tmp/pids/server.pid; true

# copy over application
ADD . $APP_HOME
{% endhighlight %}

Having that we can take care of `docker-compose.yml`

{% highlight yaml %}
db:
  image: postgres:9.4.4

redis:
  image: redis:3.0.2

web:
  build: .
  command: rails s -b 0.0.0.0 -p 3000
  volumes:
    - .:/app/rails_application
  ports:
    - 'localhost:3000:3000'
  links:
    - db
    - redis
{% endhighlight %}

Each top level section defines separate container. 2nd level are docker parameters so for `web` container we specify command to issue, mount volume, expose port (bonus points for being able to specify hostname/ip) and link with other containers.

As you can see if we remove new parts (linking and other containers) we are pretty much doing same things like command from previous section.

To spin up this configuration all we need is to execute `docker-compose up` and off we go because

* if there is no image present locally composer will download it
* if there was changes in images composer will update them
* it there was changes in `Dockerfile` composer will rebuild it

![Docker compose in action](/assets/docker-compose-in-action.png)

# The Another But

Of course there will be some issues. There is always something. So things worth considering:

* is data persistent
* how to scale this solution
* deployment process
* what will happen when images will be updated
* configuration for multiple host OS (is this needed?)
* duplication of environment variables and how to reuse them
* gems caching (rails specific)
* testing and automation (rails specific, guard)
