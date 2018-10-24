---
group: cloud-guide
title: Docker development environment
functional_areas:
  - Cloud
  - Setup
  - Configuration
---

{{site.data.var.ece}} provides a Docker environment option for those who use their local environment for development, test, or automation tasks. The {{site.data.var.ece}} Docker environment requires three, essential components: a {{site.data.var.ee}} v2 template, Docker Compose, and {{site.data.var.ece}} `{{site.data.var.ct}}` package.

## Container architecture

The [Magento Cloud Docker repository](https://github.com/magento/magento-cloud-docker) contains build information for the following Docker images:

-  **DB**  
    Database based on MariaDB version 10
-  **FPM**—[magento/magento-cloud-docker-php](https://hub.docker.com/r/magento/magento-cloud-docker-php)  
    PHP-CLI: version 7 and later  
    PHP-FPM: version 7 and later  
-  **NGINX**—[magento/magento-cloud-docker-nginx](https://hub.docker.com/r/magento/magento-cloud-docker-nginx)  
    Web server based on NGINX version 1.9
-  **Redis**—[magento/magento-cloud-docker-redis](https://hub.docker.com/r/magento/magento-cloud-docker-redis)  
    Redis server based on the latest Redis version
-  **Varnish**—[magento/magento-cloud-docker-varnish](https://hub.docker.com/r/magento/magento-cloud-docker-varnish)  
    Based on the latest Varnish version and used for caching
-  **RabbitMQ**—[rabbitmq](https://hub.docker.com/_/rabbitmq)  
    Based on the latest official RabbitMQ version
-  **ElasticSearch**—[magento/magento-cloud-docker-elasticsearch](https://hub.docker.com/r/magento/magento-cloud-docker-elasticsearch)  
    Based on version 5.2

### Web container

The web container works with the [PHP-FPM](https://php-fpm.org) to serve PHP code, the **DB** image for the local database, and the **Varnish** image to send requests and cache the results.

### CLI container

The CLI container is based on a PHP-CLI image that provides `magento-cloud` and `{{site.data.var.ct}}` commands and performs file system operations. The CLI container depends on the **DB** image for the local database and the **Redis** image.

-  `build`—extends the CLI container to perform operations with writable filesystem, similar to the build phase
-  `cron`—extends the CLI container to run cron

    -  The `setup:cron:run` and `cron:update` commands are not available on Cloud and Docker for Cloud environment
    -  Cron only works with CLI container to run `./bin/magento cron:run` command

-  `deploy`—extends the CLI container to use read-only file system, similar to the deploy phase

As an example, to run the `{{site.data.var.ct}}` ideal-state command:

```bash
docker-compose run cli ece-command wizard:ideal-state
...
 - Your application does not have the "post_deploy" hook enabled.
The configured state is not ideal
```

### Cron container

The Cron container is based on PHP-CLI images, and executes operations in the background immediately after the Docker environment start.

#### View cron log

```bash
docker-compose run cli bash -c "cat /var/www/magento/var/log/magento.cron.log"
```

### Database container

The database container is based on the `mariadb:10` image.

#### Importing a database dump

To import a database dump, place the SQL file into the `docker/mysql/docker-entrypoint-initdb.d` folder. The `{{site.data.var.ct}}` package imports and processes the SQL file the next time you build and start the Docker environment using the `docker-compose up` command.

Although it is a more complex approach, you can use GZIP by _sharing_ the `.sql.gz` file using the `docker/mnt` directory and importing it inside the Docker container.

## Sharing data between host machine and container

You can easily share files between your machine and a Docker container by placing the files in the `docker/mnt` directory. They appear in `/mnt` directory the next time you build and start the Docker environment using the `docker-compose up` command. 

## Docker commands

Action | Command
:--- | :---
Build and start Docker environment | `docker-compose up -d`
Build environment | `docker-compose run build cloud-build`
Deploy environment | `docker-compose run deploy cloud-deploy`
Connect to CLI container | `docker-compose run cli bash`
Use `{{site.data.var.ct}}` command | `docker-compose run cli ece-command <command>`
Use Magento CLI command | `docker-compose run cli magento-command <command>`
Stop and remove Docker environment (removes volumes) | `docker-compose down -v`
Stop Docker environment without destroying containers | `docker-compose stop`
Resume Docker environment | `docker-compose start`
{:style="table-layout:auto;"}

## Automate integration testing

Installing {{site.data.var.ece}} in a dedicated Docker environment presents an opportunity for you to customize the following features and capabilities to implement automated integration testing:

-  2-layer configuration for Docker build and Travis CI
-  Customizable Docker Compose file to support a wide range of environment sets
-  Flexible permissions
-  Easy local setup and implementation
