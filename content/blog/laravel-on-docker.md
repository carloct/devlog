+++
categories = ["development"]
date = "2018-09-11T03:37:00+01:00"
description = "No boilerplate, just the minimum"
tags = ["laravel", "docker"]
title = "Minimal Docker setup for Laravel on macOS"
type = "post"

+++
There are many pre-built solutions for setting up a Laravel + Docker environment, they're all excellent solutions, but too complex for my needs 90% of the times.

It' s the _boilerplate syndrome_, a starter package that is supposed to ease the pain of creating a new project but then soon becomes a huge blob that tries to suit every possible setup.

I wanted something that would let me start writing code with a minimal footprint, that could be eventually improved if necessary.

#### Install Docker for Mac

Download the [Docker for Mac](https://www.docker.com/community-edition#/download)

Recently Docker split the whole project in different releases, the Community Edition (free) is what you want.

Follow the instructions, this will install Docker on your local machine

#### Create a dev stack with Docker Compose

Docker is great for running single processes, but in most cases, you will need more than a single for running a full Laravel application.

Docker offers an easy way to define a stack of services called Docker Compose. It's a simple YAML file where you define the containers (aka services) you need.

In particular, we need:

* Nginx
* Php 7.1 as FPM
* Mysql

Create a folder, I'll name it `docker-laravel`, choose any name you like.

```bash
mkdir docker-laravel
cd docker-laravel
```

Create the Doker compose file

```bash
vi docker-compose.yml
```

#### Nginx

Let's define the Nginx container

    version: '3'
    services:
        nginx:
            image: nginx:latest
            ports:
                - "80:80"

* `version` is the docker compose syntax version, in this case, the version 3, the latest at the time of writing
* `services` is the list of services you want to define
* `nginx` is the name you choose for the service, can be anything
* `image` is the docker image that you want to use for the service, in this case, the official [Nginx image](https://hub.docker.com/_/nginx/) published on Dockerhub (a central repository for public images)
* `ports` are the ports you want to map to the service. We're mapping the internal docker container port 80, the local machine port 80

That's it, you have defined a stack with a single container, try it running:

```bash
docker-compose up
```

Docker will download the Nginx image, create the container and expose the ports.
If you go to `localhost:80` in your browser you will see the default Nginx page

![](/uploads/welcome_to_nginx.png)

#### PHP

The setup for the PHP container will require some tweaking since the official [PHP image](https://hub.docker.com/_/php/) doesn't come with all the extensions we need to run Laravel. We will _extend_ the official image and add everything we need.

Since also the Nginx image will require some additional command, we can keep all the customised images in a sub-folder `image`, within `docker-laravel`.

This will be the final layout:

    docker-laravel
    |
    |-images
      |-php-fpm
      |-nginx
    docker-compose.yml

Create the `php-fpm` folder

    mkdir -p  images/php-fpm
    cd images/php-fpm

We can extend the official Php image by creating a Dockerfile

    vi Dockerfile

Dockerfile

    FROM php:fpm
    
    RUN apt-get update && apt-get install -y \
        libmcrypt-dev && \
        docker-php-ext-install pdo_mysql mcrypt

* `FROM` defines the image we're extending from
* `RUN` is the command that the container will execute when started, in this case, we install `mcrypt` and enable it with the prebuilt command `docker-php-install` command. We enable also `pdo_mysql`

Now we can use this image in our `docker-compose.yml`

    version: '3'
    services:
        nginx:
            image: nginx:latest
            ports:
                - "80:80"
        fpm:
            build: ./images/php-fpm
            ports:
                - "9000:9000"

The container is called `fpm`, you can choose anything you like.
Instead of defining an `image`, we use `build` that simply tells docker that it needs to build the image from the Dockerfile in `./images/php-fpm`

Now we have Nginx and Php... in a black box, how can we share files from the host machine to the docker containers?

Docker compose let us specify a _volume_ we want to share with the containers, basically a shared folder from the host machine to the containers.

    version: '3'
    services:
        nginx:
            image: nginx:latest
            ports:
                - "80:80"
            volumes:
                - ./:/var/www/laravel
            working_dir:
                /var/www/laravel
        fpm:
            build: ./images/php-fpm
            ports:
                - "9000:9000"
            volumes:
                - ./:/var/www/laravel
            working_dir:
                /var/www/laravel

`volume` will share the current directory (the directory where the docker-compose.yml file is) with a directory `/var/www/laravel`.

The syntax is `local_folder:container_folder`

The folder is shared with both Nginx and Php containers, this is not necessary, but having the source code mapped also to the Php container will ease the pain of running Composer.

`working_dir` is just a helper to tell which folder we want to be the default folder.

We have the source code, but Nginx doesn't know anything about it and doesn't know about the Php container, the two containers are completely isolated, we need to make them see each other and define a virtual host for Nginx.

#### Links

With the directive `link`, we can define a link between containers.

    version: '3'
    services:
        nginx:
            image: nginx:latest
            ports:
                - "80:80"
            links:
                - fpm
            volumes:
                - ./:/var/www/laravel
            working_dir:
                /var/www/laravel
        fpm:
            build: ./images/php-fpm
            ports:
                - "9000:9000"
            volumes:
                - ./:/var/www/laravel
            working_dir:
                /var/www/laravel

Now Nginx can _see_ the Php container. We defined the Php container as `fpm` so we need to use the same name to link it up.

#### Nginx virtual host

Nginx needs to know about the source code we shared with its container.
Create a `vhost.conf` file in `images/nginx`

    server {
        listen 80;
        server_name laravel.dev;
        root /var/www/laravel/public;
    
        index index.html index.htm index.php;
    
        charset utf-8;
    
        location / {
            try_files $uri $uri/ /index.php$is_args$args;
        }
    
        location = /favicon.ico { access_log off; log_not_found off; }
        location = /robots.txt  { access_log off; log_not_found off; }
    
        access_log off;
        error_log  /var/log/nginx/Laravel-error.log error;
    
        sendfile off;
    
        client_max_body_size 100m;
    
        location ~ \.php$ {
            fastcgi_split_path_info ^(.+\.php)(/.+)$;
            fastcgi_pass fpm:9000;
            fastcgi_index index.php;
            include fastcgi_params;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            fastcgi_intercept_errors off;
            fastcgi_buffer_size 16k;
            fastcgi_buffers 4 16k;
        }
    
        location ~ /\.ht {
            deny all;
        }
    }

And share this file with the nginx container

    volumes:
        - ./images/nginx/vhost.conf:/etc/nginx/conf.d/laravel.conf

to make things easier, map the virtual host in your local `hosts` file

    vi /etc/hosts
    
    ##
    # Host Database
    #
    # localhost is used to configure the loopback interface
    # when the system is booting.  Do not change this entry.
    ##
    127.0.0.1       localhost
    255.255.255.255 broadcasthost
    ::1             localhost
    
    127.0.0.1       laravel.dev

#### Mysql

The last step is Mysql. Luckily the official image will do just fine this time. Let's define the service in `docker-compose`

    version: '3'
    services:
    
        [...]
    
        db:
            image: mysql:5.7
            ports:
                - "3360:3360"
            volumes:
                - data:/var/lib/mysql
            environment:
                MYSQL_ROOT_PASSWORD: password
                MYSQL_DATABASE: laravel
                MYSQL_USER: laravel
                MYSQL_PASSWORD: time3483
    
    volumes:
        data:

The `volumes` directive specify a `data volume`, a persistent volume that won't be destroyed when you stop the container, so your Mysql database won't be lost.

With `environment` you can define default values for the database you will create, root password, a default database, a user, and a user password.

The last thing we need to do is to link the db container to Php

    [...]
    fpm:
        build: ./images/php-fpm
        ports:
            - "9000:9000"
        links:
            - db
        volumes:
            - ./:/var/www/laravel
        working_dir:
            /var/www/laravel
    [...]

The final `docker-compose` will look like this:

    version: '3'
    services:
        nginx:
            image: nginx:latest
            ports:
                - "80:80"
            links:
                - fpm
            volumes:
                - ./:/var/www/laravel
                - ./images/nginx/vhost.conf:/etc/nginx/conf.d/laravel.conf
            working_dir:
                /var/www/laravel
        fpm:
            build: ./images/php-fpm
            ports:
                - "9000:9000"
            links:
                - db
            volumes:
                - ./:/var/www/laravel
            working_dir:
                /var/www/laravel
    
        db:
            image: mysql:5.7
            ports:
                - "3360:3360"
            volumes:
                - data:/var/lib/mysql
            environment:
                MYSQL_ROOT_PASSWORD: password
                MYSQL_DATABASE: laravel
                MYSQL_USER: laravel
                MYSQL_PASSWORD: time3483
    
    volumes:
        data:

Run `docker-compose up` and the images will be downloaded/built and the containers will be started. You can access it via `laravel.dev` in your browser.

Now your app lives within a container, in particular, the php process will have access to the database via the `db` link, this is something to keep in mind when running migrations since the `.env` file will need to have the correct database host

    DB_HOST=db