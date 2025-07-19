---
layout: post
title: Adding external services to a Ruby on Rails project with docker-compose
description: Extend your dockerized Rails application by adding PostgreSQL database and web servers using docker-compose for a complete development environment.
---

The first part in this series went over the basics of setting up a new Rails app using Docker. Part one also showed how to leverage `docker-compose` in setting up your application to be run. This second part of the series will take a deeper look into taking `docker-compose` further to architect a more complex Rails application. Most Rails applications utitlize several external services to augment the core Rails service. Follow along to see how to add databases and web servers to create a more complete package.

## Adding Postgresql
As with most Rails applications, you’ll need to store your data somewhere. With Docker, adding any database you want becomes trivial. Since you define services within your `docker-compose.yml` file, you won’t have to worry too much about installing databases manually. Postgresql is usually a popular choice when it comes to Rails and databases. You can add a Postgresql server to you application by including the following to the bottom of `docker-compose.yml`.

```
  db:
    image: postgres
    environment:
      - POSTGRES_PASSWORD=postgres
    volumes:
      - /var/lib/postgresql/data	
```

The next time you start up your application (`docker-compose up`), this will tell docker compose to create a new service called `db` and use the image called `postgres`. This `db` service also has a volume mounted. This means that the directory specified by `/var/lib/postgresql/data` will *persist* if or when the service container gets removed. This directory happens to be where Postgres stores it data.

## Interacting with Postgresql
Having direct access to your database is really useful during development. Here are two ways you can interact with your Postgres database.

Start a `psql` console by running the following command from your host computer:

```
docker-compose exec db psql -U postgres postgres
```

This will start a command line interface to Postgres. With this. you can issue SQL statements directly to the database server. This command uses the `exec` docker command, which differs slightly from `run`. When commands are executed with `exec` they do not start up a separate container process, but instead run within the context of an already running contianer. This allows the `psql` command to connect to the socket file that the actual Postgres server is running on in its container.

If you get an error saying that a container cannot be found:

```
ERROR: No container found for db_1
```

That means you need to start at least the `db` service by running:

```
docker-compose up db
```

When the `db` container starts up, it will create the database and a user `postgres` with the password specified, `postgres`.

If you use GUI tools to interact with databases, the `db` service will need a bit more configuration. Since the database server is running in the context of a container, your GUI tool isn’t exactly able to see it directly. To facilitate a connection, add the following configuration to your `docker-compose.yml` file:

```
    db:
      image: postgres
      environment:
        - POSTGRES_PASSWORD=postgres
+     ports:
        - 15432:5432
      volumes:
        - /var/lib/postgresql/data
```

By default, Postgres runs on port `5432`. This means that an application may connect to Postgres on that port number. However, Docker containers run within their own isolated network infrastructure. By defining a host port in the `docker-compose.yml` file, you are telling Docker to “bind” the `db` container’s port of `5432`, to the host port of `15432`. Now with your GUI tool, you can connect to your localhost on port `15432`.

## Connecting from Rails
Now that you have a Postgres database running, you can configure Rails to use it as the database. For the purposes of demonstration, connect to the db under the development environment. This is accomplished by modifying your `config/database.yml` file in your Rails project directory. Look for the `development` entry and modify it as follows:

```
development:
  adapter: postgresql
  host: db
  username: postgres
  password: postgres
```

The host `db` maps to the `db` service defined in `docker-compose.yml`. The neat thing here is that docker-compose has it’s own internal DNS system which you can use to refer to anywhere in the context of a container.

## Adding Redis
Adding a Redis instance to your architecture follows much of the same path. Do this by adding the following changes to `docker-compose.yml`:

```
  redis:
    image: redis
    ports:
      - 16379:6379
```

Start the redis service with the `docker-compose` command:

```
docker-compose up redis
```

This will start the redis server binding the container to ports `16379` on the host machine.

## Interacting with Redis
Now you can also start interacting with Redis using tools you are already familiar with. If you have any GUI tools, you can connect to your local port of `16379`. If you want to open a comand line redis prompt, use the following:

```
docker-compose run --rm redis redis-cli -h redis
```

This docker command issues a new redis container and starts the `redis-cli` command line tool, passing the host argument `-h redis`. You’ll notice that this host matches the name of the service defined in your `docker-compose.yml` file. Docker compose provides this internal host name lookup as part of its networking infrastructure. So anytime you need to refer to one of your services within your system, you can use its service name.

## Connecting from Rails
Similar to Postgres, you may need some sort of configuration in Rails to connect to Redis. Depending on your specific need, this differs. Most uses of Redis just requires the use of a `REDIS_URL` environment variable. In these cases, you can add this as part of your `docker-compose.yml` file:

```
  app:
    environment:
      - REDIS_URL=redis://redis:6379
```

You’ll notice that this URL is made up of the `redis` hostname, as well as the port `6379` as configured in the `redis` service.

## Adding Nginx
An integral part to any web application is going to be the web server. Although the default web server shipped with Rails, Puma, can handle web requests on it’s own, it’s often not enough with any real world load. A dedicated web server helps queue web requests as they arrive so that they can be processed by your Rails application. These web servers are made to handle high loads and effectively manage the amount of traffic that gets processed by Rails.

Adding Nginx can be added easiliy, however, configuring it is sometimes a daunting task. You’ll find that the easiest way to setup Nginx is mounting the configuration directory with our service definition. Add the following to your `docker-compose.yml` file:

```
  web:
    image: nginx
    volumes:
      - ./nginx/conf.d:/etc/nginx/conf.d
    links:
      - app
    ports:
      - 80:80
```

Try starting up your application. The nginx service should start up just fine, but won’t actually do anything. You’ll need to do a bit of configuration to tell it how to respond to requests and where to find your Rails application.	

## Configuring Nginx
There is a lot to know about confguring an Nginx web server, however, in this section, you’ll only need to go over some of the basics. First you’ll need to create a configuration file that’ll go in a directory in your project folder under `nginx/conf.d`. You’ll notice that this directory gets mounted to `/etc/nginx/conf.d` in the Docker container. This is a special (default) location that Nginx will look in for configuration files. Create a file here and call it `rails.conf` with the following contents:

```
server {
  listen      80;
  listen [::]:80;
  server_name _;

  location / {
    proxy_pass http://app:3000;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_redirect off;
  }
}
```

Restart your docker-compose application. Once everything is up and running, you should be able to navigate to http://localhost in a browser. This should send a request to Nginx which then gets proxied to your Rails application. In the above configuration you’ll notice that Nginx is listening on port `80`, this should correspond to the “internal port mapping” in your `docker-compose.yml` file port configuration for the `web` service. The second port is the internal one. The `proxy_pass` directive is telling Nginx to forward to a host named `app`, which is the named service for your Rails app. One neat thing here is that internal DNS for  `docker-compose` comes with it’s own load balancing techniques. You can scale up your `app` service to run multiple containers of the service within `docker-compose`. Ngnix will continue proxying requests to the `app` service, but `docker-compose` will take care of balancing the requests between any running containers of the `app` service!

## What’s Next
This tutorial should give you a brief introduction into how add more services to your application architecture. Having everything defined in your `docker-compose.yml` file makes it easy to keep track of your Rails application dependencies. This makes it easier to reproduce development environments for yourself and your teammates.

The next series in this tutorial will show you what it takes to take your application to production. You’ll see how setting up your architecture in `docker-compose` translates into a production environment. By leveraging a development environment that closely resembles productions goes a long way in making changes easier. Stay tuned!
