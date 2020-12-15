---
layout: post
title: Creating a new Ruby on Rails project with docker-compose
---

There’s nothing like starting with a clean slate on a new Ruby on Rails project. Creating a new project on the latest version of Ruby and Rails is a great way to keep your skills sharp. Using docker and docker-compose to create a new Rails app has never been simpler. In addition, you are defining your infrastructure from the start. This will ensure a smoother transition to production, and increase maintainability of your project.

Follow along you’ll learn how to leverage docker and learn a few of the basic commands to get started. You’ll get a brief theory behind containerization while at the same time starting a new Rails project with some of the most common features.

## Getting Started
Head over to Docker’s website, [Get Started with Docker | Docker](https://www.docker.com/get-started), and locate the “Docker Desktop” application download. Select the option that matches your computer. Save the file and initiate the install by double clicking on the downloaded file. The exact install process will differ depending on your OS. Go with any default options when asked. Once your install is complete, the Docker engine should automatically startup on your computer.

While following the rest of the tutorial, make sure to create a new project directory where you’ll be issuing commands and creating files and folders for your project.

## Basic Docker Commands
There is a lot to learn about Docker, however, learning a few of the basic commands will build a good foundation. These commands will be used often whenever you are working with Docker, so don’t worry if you need to refer back here often.

You’ll be working with Ruby when working with Rails, so start by downloading the *image* that will run the Ruby interpreter. An *image* in the world of containers is basically a snapshot of a file system. This command will download the offical Ruby image from docker to your computer:

```
docker pull ruby
```

This will download the latest Docker image for the image called `ruby`. This image is uses Debian as its base image. You can view more details about this image, as well other available versions on the Docker Hub website, [Docker Hub](https://hub.docker.com/_/ruby).  Once complete, you should now be able to start running the ruby commands. Try the following command to verify the version you have just downloaded:

```
docker run ruby ruby -v
```

This command will start a new Docker container using the `ruby` image. There is quite a bit going on here, so here is a breakdown of the command you just run:

* `docker` - the application you are telling your OS to run
* `run` - the command passed to Docker, which tells the application to create a new container
* `ruby` - the name of the image of which the container will be created from
* `ruby -v` - the command passed which is executed in the context of the container

The command should return and output the ruby version that is being run. If you’ve gotten this far, you are in good shape.

## Create the docker-compose.yml file
Now that you have the ability to run Docker containers locally, it’s time to start setting up your project. Create a file called `docker-compose.yml` in your project directory with the following contents:

```
services:
  app:
    image: ruby:alpine
    volumes:
      - .:/app
```

This `docker-compose.yml` file describes your applications services. To keep things simple, you’ve defined just a single service called `app`. This service uses the `ruby:alpine` image. The `alpine`  tag is a Linux variant that is more lightweight than Debian. The `volume` setting mounts a single volume. This means that when the `app` container starts, the `/app` directory in the container will point directly to `.` or the current directory of where the `docker-compose.yml` file is defined.

You can explore what this service looks like by starting a `sh` shell in the context of the container. Run the following command:

```
docker-compose run --rm app sh
```

You should get a shell prompt. Change your directory to `/app` and list the files in the directory. You should see something similar to this:

```
root@aa686e51fea8:/# cd /app
root@aa686e51fea8:/app# ls
docker-compose.yml
```

That `docker-compose.yml` file is the same file you just created in your project directory. This is what’s known as directory mapping, or volume mounting. You’ll also notice that any changes you make in the directory in the context of the container will also be seen by your OS and vice versa.

You should become familiar with being able to run commands using docker and docker-compose. As you follow along, you’ll be asked to run a command in a shell in the container, so you may need to refer to this section a few times on how to do that.

## Bootstrap the Rails gem and create a project
With the `app` service defined, you can start building that new Ruby on Rails app. Issue the following command in a shell within the container:

```
apk --update add nodejs yarn build-base ruby-dev tzdata git
gem install rails
```

This will install a few Rails dependencies, as well as the Rails gem itself in the context of your `app` service container. When it’s complete, you can  start issuing Rails commands. Create a new Rails app in your project directory in the contianer (`/app`):

```
rails new /app
```

Remember, all of this is happening within the conatiner. The rails command will start the process of creating a brand new Ruby on Rails application in the project folder. An important thing to remember is that if you exit out of the running container, you’ll lose any changes made outside of the mounted `/app` directory. This means that any gems installed while you were in shell will need to be reinstalled if you start a new shell session.

## Create the Dockerfile file
Now that we have a basic Ruby on Rails app, we can start defining a `Dockerfile` that will describe how to “build” it. Building is the process of creating an image. This file will contain the instructions Docker will use to setup an image capable of running your Ruby on Rails application.

Create a new file in your project directory called `Dockerfile` with the following contents:

```
FROM ruby:alpine

RUN apk --update add nodejs yarn build-base ruby-dev tzdata git

RUN mkdir /app
WORKDIR /app

ADD Gemfile /app/Gemfile
ADD Gemfile.lock /app/Gemfile.lock

RUN bundle install

ADD package.json /app/package.json
ADD yarn.lock /app/yarn.lock

RUN yarn install

ADD . /app

RUN bin/rails assets:precompile
```

Some of this will look very familiar since they were used to bootstrap creating a new Rails app previously. This `Dockerfile` will create an image based on the official Docker  `ruby:alpine`  image. It will then add some of the prequisite OS packages, ruby gems, and npm packages.

## Update the docker-compose.yml
To make use of the custom image you will be using, specify the app service to build it. This is done by switching out the `image` property and replacing it with a `build` property in the `docker-compose.yml` file. The change looks like this:

```
 services:
   app:
-    image: ruby
+    build: .
     volumes:
       - .:/app
```

Now when docker-compose run this application, it will “build” the image based on the context of the current project directory. By default, the process will look for the `Dockerfile` file. You can start the build process by running the following command on your host machine:

```
docker-compose build
```

The docker-compose process will then scan all the defined services in your `docker-compose.yml` file and run the build process for each. This build process can sometimes take a while on first go, but subsequent builds use caching to speed things up.

Now that you verified that your application can now build, you will need to tell Docker how to start your application. This is done by adding a `command`  and `ports` directive to your `app` service in the `docker-compose.yml` file.

```
services:
   app:
     build: .
+    command: bin/rails server -b 0.0.0.0 -p 3000
+    ports:
+      - 3000
     volumes:
       - .:/app
```

If you are already familiar with Ruby on Rails, the command value will look familiar to you. This command will start the Ruby on Rails server process, binding to the `0.0.0.0` ip address of the container while listening on port `3000` . The `ports` docker-compose directive forwards the port from your host machine, to the running Docker container service. With this in place, you should now be able to start your application. Run the following from your host machine:

```
docker-compose up
```

Docker will now ensure that all services are built, create a container for your `app` service, and start the `command` specified. Once you see the line `app_1  | * Listening on tcp://0.0.0.0:3000`, your applcation should now be available in a browser by visiting the following link: [http://localhost:3000](http://localhost:3000).  If all went well, you should see the Ruby on Rails welcome screen. At this point you now have a Rails app running in a docker container.

## Running Rails commands
Ruby on Rails comes with a lot of tools to aid in development. Many of these tools need to be run in a console. With Docker, it’s easy to run any command you need, take a look at an example:

```
docker-compose run --rm app bin/rails console
```

Executing the above command will start a rails console within the context of the Docker container `app`. The `--rm` switch tells Docker to remove the container after you exit out of the console.

## What’s Next
Congrats on making it this far! It’s a lot to take in, so refer back to this page as necessary. Learning how to use Rails with Docker may seem daunting at first. Keep at it. One you start becoming familiar with using Docker, a lot of it will start to make sense. Hopefully this tutorial has helped clear things up for you. There is still a lot more to cover. (Stay tuned for the second part of this series. When you are ready, head on over to the second part of this series here.)
