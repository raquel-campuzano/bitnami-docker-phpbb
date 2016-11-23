[![CircleCI](https://circleci.com/gh/bitnami/bitnami-docker-phpbb/tree/master.svg?style=shield)](https://circleci.com/gh/bitnami/bitnami-docker-phpbb/tree/master)
[![Docker Hub Automated Build](http://container.checkforupdates.com/badges/bitnami/phpbb)](https://hub.docker.com/r/bitnami/phpbb/)
# What is phpBB?

> If you need to build a community forum, try phpBB. First released in 2000, phpBB is a bulletin board solution that allows you to create forums and subforums. phpBB supports the notion of users and groups, file attachments, full-text search, notifications and more. Hundreds of modifications are available including themes, communications add-ons, spam management and more.

https://www.phpbb.com/

# Prerequisites

To run this application you need Docker Engine 1.10.0. Docker Compose is recomended with a version 1.6.0 or later.

# How to use this image

## Run phpBB with a Database Container

Running phpBB with a database server is the recommended way. You can either use docker-compose or run the containers manually.

### Run the application using Docker Compose

This is the recommended way to run phpBB. You can use the following docker compose template:

```yaml
version: '2'

services:
  mariadb:
    image: 'bitnami/mariadb:latest'
    volumes:
      - 'mariadb_data:/bitnami/mariadb'
  application:
    image: 'bitnami/phpbb:latest'
    ports:
      - '80:80'
      - '443:443'
    volumes:
      - 'phpbb_data:/bitnami/phpbb'
      - 'apache_data:/bitnami/apache'
      - 'php_data:/bitnami/php'
    depends_on:
      - mariadb

volumes:
  mariadb_data:
    driver: local
  phpbb_data:
    driver: local
  apache_data:
    driver: local
  php_data:
    driver: local
```

### Run the application manually

If you want to run the application manually instead of using docker-compose, these are the basic steps you need to run:

1. Create a new network for the application and the database:

  ```bash
  $ docker network create phpbb_network
  ```

2. Start a MariaDB database in the network generated:

  ```bash
  $ docker run -d --name mariadb --net=phpbb_network bitnami/mariadb
  ```

  *Note:* You need to give the container a name in order to phpBB to resolve the host

3. Run the phpBB container:

  ```bash
  $ docker run -d -p 80:80 --name phpbb --net=phpbb_network bitnami/phpbb
  ```

Then you can access your application at http://your-ip/

## Persisting your application

If you remove every container and volume all your data will be lost, and the next time you run the image the application will be reinitialized. To avoid this loss of data, you should mount a volume that will persist even after the container is removed. If you are using docker-compose your data will be persistent as long as you don't remove `mariadb_data` and `application_data` data volumes. If you have run the containers manually or you want to mount the folders with persistent data in your host follow the next steps:

> **Note!** If you have already started using your application, follow the steps on [backing](#backing-up-your-application) up to pull the data from your running container down to your host.

### Mount persistent folders in the host using docker-compose

This requires a sightly modification from the template previously shown:

```yaml
version: '2'

services:
  mariadb:
    image: 'bitnami/mariadb:latest'
    volumes:
      - '/path/to/your/local/mariadb_data:/bitnami/mariadb'
  application:
    image: 'bitnami/phpbb:latest'
    ports:
      - '80:80'
      - '443:443'
    volumes:
      - '/path/to/your/local/phpbb_data:/bitnami/phpbb'
      - '/path/to/your/local/apache_data:/bitnami/apache'
      - '/path/to/your/local/php_data:/bitnami/php'
    depends_on:
      - mariadb
```

### Mount persistent folders manually

In this case you need to specify the directories to mount on the run command. The process is the same than the one previously shown:

1. If you haven't done this before, create a new network for the application and the database:

  ```bash
  $ docker network create phpbb_network
  ```

2. Start a MariaDB database in the previous network:

  ```bash
  $ docker run -d --name mariadb -v /your/local/path/bitnami/mariadb_data:/bitnami/mariadb  --net=phpbb_network bitnami/mariadb
  ```

  *Note:* You need to give the container a name in order to phpBB to resolve the host

3. Run the phpBB container:

  ```bash
  $ docker run -d -p 80:80 --name phpbb -v /your/local/path/bitnami/phpbb:/bitnami/phpbb --net=phpbb_network bitnami/phpbb
  ```

# Upgrade this application

Bitnami provides up-to-date versions of MariaDB and phpBB, including security patches, soon after they are made upstream. We recommend that you follow these steps to upgrade your container. We will cover here the upgrade of the phpBB container. For the MariaDB upgrade see https://github.com/bitnami/bitnami-docker-mariadb/blob/master/README.md#upgrade-this-image

1. Get the updated images:

  ```bash
  $ docker pull bitnami/phpbb:latest
  ```

2. Stop your container

 * For docker-compose: `$ docker-compose stop phpbb`
 * For manual execution: `$ docker stop phpbb`

3. (For non-compose execution only) Create a [backup](#backing-up-your-application) if you have not mounted the phpbb folder in the host.

4. Remove the currently running container

 * For docker-compose: `$ docker-compose rm -v phpbb`
 * For manual execution: `$ docker rm -v phpbb`

5. Run the new image

 * For docker-compose: `$ docker-compose start phpbb`
 * For manual execution ([mount](#mount-persistent-folders-manually) the directories if needed): `docker run --name phpbb bitnami/phpbb:latest`

# Configuration
## Environment variables
 When you start the phpbb image, you can adjust the configuration of the instance by passing one or more environment variables either on the docker-compose file or on the docker run command line. If you want to add a new environment variable:

 * For docker-compose add the variable name and value under the application section:

```yaml
application:
  image: bitnami/phpbb:latest
  ports:
    - 80:80
  environment:
    - PHPBB_PASSWORD=my_password
```

 * For manual execution add a `-e` option with each variable and value:

```bash
 $ docker run -d -e PHPBB_PASSWORD=my_password -p 80:80 --name phpbb -v /your/local/path/bitnami/phpbb:/bitnami/phpbb --net=phpbb_network bitnami/phpbb
```

Available variables:

 - `PHPBB_USERNAME`: phpBB application username. Default: **user**
 - `PHPBB_PASSWORD`: phpBB application password. Default: **bitnami**
 - `PHPBB_EMAIL`: phpBB application email. Default: **user@example.com**
 - `MARIADB_USER`: Root user for the MariaDB database. Default: **root**
 - `MARIADB_PASSWORD`: Root password for the MariaDB.
 - `MARIADB_HOST`: Hostname for MariaDB server. Default: **mariadb**
 - `MARIADB_PORT`: Port used by MariaDB server. Default: **3306**

### SMTP Configuration

To configure phpBB to send email using SMTP you can set the following environment variables:
 - `SMTP_HOST`: SMTP host.
 - `SMTP_PORT`: SMTP port.
 - `SMTP_USER`: SMTP account user.
 - `SMTP_PASSWORD`: SMTP account password.
 - `SMTP_PROTOCOL`: SMTP protocol.

This would be an example of SMTP configuration using a GMail account:

 * docker-compose:

```yaml
  application:
    image: bitnami/phpbb:latest
    ports:
      - 80:80
    environment:
      - SMTP_HOST=smtp.gmail.com
      - SMTP_PORT=587
      - SMTP_USER=your_email@gmail.com
      - SMTP_PASSWORD=your_password
```

 * For manual execution:

```bash
 $ docker run -d -e SMTP_HOST=smtp.gmail.com -e SMTP_PORT=587 -e SMTP_USER=your_email@gmail.com -e SMTP_PASSWORD=your_password -p 80:80 --name phpbb -v /your/local/path/bitnami/phpbb:/bitnami/phpbb --net=phpbb_network bitnami/phpbb
```


# Backing up your application

To backup your application data follow these steps:

1. Stop the running container:

  * For docker-compose: `$ docker-compose stop phpbb`
  * For manual execution: `$ docker stop phpbb`

2. Copy the phpBB data folder in the host:

  ```bash
  $ docker cp /your/local/path/bitnami:/bitnami/phpbb
  ```

# Restoring a backup

To restore your application using backed up data simply mount the folder with phpBB data in the container. See [persisting your application](#persisting-your-application) section for more info.

# Contributing

We'd love for you to contribute to this container. You can request new features by creating an
[issue](https://github.com/bitnami/bitnami-docker-phpbb/issues), or submit a
[pull request](https://github.com/bitnami/bitnami-docker-phpbb/pulls) with your contribution.

# Issues

If you encountered a problem running this container, you can file an
[issue](https://github.com/bitnami/bitnami-docker-phpbb/issues). For us to provide better support,
be sure to include the following information in your issue:

- Host OS and version
- Docker version (`docker version`)
- Output of `docker info`
- Version of this container (`echo $BITNAMI_IMAGE_VERSION` inside the container)
- The command you used to run the container, and any relevant output you saw (masking any sensitive
information)

# License

Copyright (c) 2016 Bitnami

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
