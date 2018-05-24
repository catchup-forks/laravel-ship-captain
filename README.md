# Laravel Ship Captain
A repository containing Docker containers and configs for basic Laravel development & production environment.

## How to build & run
You just have to build the images:

```bash
$ docker-compose build
```

After building the images, you can create the containers:

```bash
$ docker-compose up -d
```

## Available containers
* `base`: Linux environment with basic packages installed.
* `php-fpm`: PHP 7.2 environment which can run PHP apps.
* `consul`, `registrator`, `haproxy`: Used for load balancing across many `php-fpm` containers.
* `cadvisor`: Metrics about hardware.
* `supervisord`: PHP 7.2 environment for Supervsirod.
* `nginx`: Proxy to handle the requests.
* `mysql`: MySQL 5.7 for SQL database.
* `redis`: Redis server for Queues & in-memory storage.
* `phpmyadmin`: SQL control panel.
* `elasticsearch`: Latest ElasticSearch server to optimize searching.s
* `laravel-echo-server`: Laravel Echo Server for broadcasting.

## Features
* PHP 7.2 - it's time for an upgrade.
* Configurable PHP and MySQL.
* Proxy & Load Balacing across `php-fpm` containers.
* Crontab already set (cron running every minute)
* Laravel Echo Server pre-built for broadcasting.
* AWS load balancer service available.

## Services running on ports
* `80`: Your server
* `1936`: HAProxy stats
* `2777`: PHPMyAdmin
* `8080`: Metrics about hardware (`cadvisor` service)
* `8500`: Metrics about containers (`consul` service)

## Configure
You are free to configure it the way you want. You have env variables inside the `docker-compose.yml` and you can freely set your own PHP extensions or ports to any container.

By default, in Laravel, the `index.php` file is located in the root, in `public` directory.
If your `index.php` is somewhere else (i.e. in the root of your project), you can change it in the `docker/nginx/sites/default.conf` file:

```
...
  server_name localhost;
  root /var/www; # This can be turned into /var/www/public.
  index index.php index.html index.htm;
...
```

## Load Balancing
For load balancing, `consul`, `registrator` and `haproxy` is used. This helps balancing the requests over your `php-fpm` services.

You have just to build and run your containers. Then you have to scale the `php-fpm` service:

```bash
$ docker-compose scale php-fpm=5
```

While the containers are created, nobody can access them. When the containers were created, the users are automatically redirected to them.

This applies even backwards: when the containers are destroyed, the users can't access them. They will be redirected to up and running containers.


## Run certain services
The build command mentioned earlier makes sure you get the whole suite. In case you need just PHP & MySQL and Redis, you can easily run:

```bash
$ docker-compose build base php-fpm mysql redis
```

and then run the containers:

```bash
$ docker-compose up -d base php-fpm mysql redis
```

Note: Many services require `base` container. It's better to build it to make sure your environment with all packages and extensions are up-to-date.
The `base` container can help you connect remotely via bash to it and run commands in the environment.

To do so, you can use `docker ps` to locate the containers' hashes and then connect to it via console:

```bash
$ docker exec -it <container_hash> bash
```

If you are on Windows, you should prefix the previous command with `winpty`:

```bash
$ winpty docker exec -it <container_hash> bash
```
## AWS Load Balancer
The `docker-compose.yml` comes with pre-configured AWS Load Balancer settings. All you have to do is uncommenting it from `docker-compose.yml` and throw your SSH keys in `docker/aws/ssh` folder.
