# What is Ghost?

> Ghost is a simple, powerful publishing platform that allows you to share your stories with the world

https://ghost.org/

# TL;DR;

## Docker Compose

```bash
$ curl -sSL https://raw.githubusercontent.com/bitnami/bitnami-docker-ghost/master/docker-compose.yml > docker-compose.yml
$ docker-compose up -d
```

# Why use Bitnami Images?

* Bitnami closely tracks upstream source changes and promptly publishes new versions of this image using our automated systems.
* With Bitnami images the latest bug fixes and features are available as soon as possible.
* Bitnami containers, virtual machines and cloud images use the same components and configuration approach - making it easy to switch between formats based on your project needs.
* All our images are based on [minideb](https://github.com/bitnami/minideb) a minimalist Debian based container image which gives you a small base container image and the familiarity of a leading linux distribution.
* Bitnami container images are released daily with the latest distribution packages available.


> This [CVE scan report](https://quay.io/repository/bitnami/ghost?tab=tags) contains a security report with all open CVEs. To get the list of actionable security issues, find the "latest" tag, click the vulnerability report link under the corresponding "Security scan" field and then select the "Only show fixable" filter on the next page.

# How to deploy Ghost in Kubernetes?

Deploying Bitnami applications as Helm Charts is the easiest way to get started with our applications on Kubernetes. Read more about the installation in the [Bitnami Ghost Chart GitHub repository](https://github.com/bitnami/charts/tree/master/upstreamed/ghost).

Bitnami containers can be used with [Kubeapps](https://kubeapps.com/) for deployment and management of Helm Charts in clusters.

# Prerequisites

To run this application you need Docker Engine 1.10.0. Docker Compose is recomended with a version 1.6.0 or later.

# Supported tags and respective `Dockerfile` links

> NOTE: Debian 8 images have been deprecated in favor of Debian 9 images. Bitnami will not longer publish new Docker images based on Debian 8.

Learn more about the Bitnami tagging policy and the difference between rolling tags and immutable tags [in our documentation page](https://docs.bitnami.com/containers/how-to/understand-rolling-tags-containers/).


* [`2-rhel-7`, `2.14.3-rhel-7-r4` (2/rhel-7/Dockerfile)](https://github.com/bitnami/bitnami-docker-ghost/blob/2.14.3-rhel-7-r4/2/rhel-7/Dockerfile)
* [`2-ol-7`, `2.14.3-ol-7-r4` (2/ol-7/Dockerfile)](https://github.com/bitnami/bitnami-docker-ghost/blob/2.14.3-ol-7-r4/2/ol-7/Dockerfile)
* [`2-debian-9`, `2.14.3-debian-9-r3`, `2`, `2.14.3`, `2.14.3-r3`, `latest` (2/debian-9/Dockerfile)](https://github.com/bitnami/bitnami-docker-ghost/blob/2.14.3-debian-9-r3/2/debian-9/Dockerfile)

Subscribe to project updates by watching the [bitnami/ghost GitHub repo](https://github.com/bitnami/bitnami-docker-ghost).

# How to use this image

### Run the application using Docker Compose

This is the recommended way to run Ghost. You can use the following docker compose template:

```yaml
version: '2'
services:
  mariadb:
    image: 'bitnami/mariadb:latest'
    environment:
      - ALLOW_EMPTY_PASSWORD=yes
      - MARIADB_USER=bn_ghost
      - MARIADB_DATABASE=bitnami_ghost
    volumes:
      - 'mariadb_data:/bitnami'
  ghost:
    image: 'bitnami/ghost:2'
    labels:
      kompose.service.type: nodeport
    environment:
      - MARIADB_HOST=mariadb
      - MARIADB_PORT_NUMBER=3306
      - GHOST_DATABASE_USER=bn_ghost
      - GHOST_DATABASE_NAME=bitnami_ghost
      - ALLOW_EMPTY_PASSWORD=yes
      - GHOST_HOST=localhost
    ports:
      - '80:2368'
    volumes:
      - 'ghost_data:/bitnami'
    depends_on:
      - mariadb
volumes:
  mariadb_data:
    driver: local
  ghost_data:
    driver: local
```

### Run the application manually

If you want to run the application manually instead of using docker-compose, these are the basic steps you need to run:

1. Create a new network for the application and the database:

  ```bash
  $ docker network create ghost-tier
  ```

2. Create a volume for MariaDB persistence and create a MariaDB container

  ```bash
  $ docker volume create --name mariadb_data
  $ docker run -d --name mariadb \
    -e ALLOW_EMPTY_PASSWORD=yes \
    -e MARIADB_USER=bn_ghost \
    -e MARIADB_DATABASE=bitnami_ghost \
    --net ghost-tier \
    --volume mariadb_data:/bitnami \
    bitnami/mariadb:latest
  ```

   *Note:* You need to give the container a name in order to Ghost to resolve the host

3. Create volumes for Ghost persistence and launch the container

  ```bash
  $ docker volume create --name ghost_data
  $ docker run -d --name ghost -p 80:80 -p 443:443 \
    -e ALLOW_EMPTY_PASSWORD=yes \
    -e GHOST_DATABASE_USER=bn_ghost \
    -e GHOST_DATABASE_NAME=bitnami_ghost \
    --net ghost-tier \
    --volume ghost_data:/bitnami \
    bitnami/ghost:latest
  ```

Access your application at http://your-ip/

> **Note!** If you want to access your application from a public IP or hostname you need to properly configured Ghost . You can handle it adjusting the configuration of the instance by setting the environment variable `GHOST_HOST` to your public IP or hostname.

## Persisting your application

If you remove the container all your data and configurations will be lost, and the next time you run the image the database will be reinitialized. To avoid this loss of data, you should mount a volume that will persist even after the container is removed.

For persistence you should mount a volume at the `/bitnami` path. Additionally you should mount a volume for [persistence of the MariaDB data](https://github.com/bitnami/bitnami-docker-mariadb#persisting-your-database).

The above examples define docker volumes namely `mariadb_data` and `ghost_data`. The Ghost application state will persist as long as these volumes are not removed.

To avoid inadvertent removal of these volumes you can [mount host directories as data volumes](https://docs.docker.com/engine/tutorials/dockervolumes/). Alternatively you can make use of volume plugins to host the volume data.

### Mount host directories as data volumes with Docker Compose

This requires a minor change to the `docker-compose.yml` template previously shown:

```yaml
version: '2'

services:
  mariadb:
    image: 'bitnami/mariadb:latest'
    environment:
      - ALLOW_EMPTY_PASSWORD=yes
      - MARIADB_USER=bn_ghost
      - MARIADB_DATABASE=bitnami_ghost
    volumes:
      - /path/to/mariadb-persistence:/bitnami
  ghost:
    image: bitnami/ghost:latest
    depends_on:
      - mariadb
    ports:
      - '80:2368'
    environment:
      - ALLOW_EMPTY_PASSWORD=yes
      - GHOST_DATABASE_USER=bn_ghost
      - GHOST_DATABASE_NAME=bitnami_ghost
      - GHOST_HOST=localhost
    volumes:
      - '/path/to/ghost-persistence:/bitnami'
```

### Mount host directories as data volumes using the Docker command line

In this case you need to specify the directories to mount on the run command. The process is the same than the one previously shown:

1. Create a network (if it does not exist):

  ```bash
  $ docker network create ghost-tier
  ```

2. Create a MariaDB container with host volume:

  ```bash
  $ docker run -d --name mariadb --net ghost-tier \
    -e ALLOW_EMPTY_PASSWORD=yes \
    -e MARIADB_USER=bn_ghost \
    -e MARIADB_DATABASE=bitnami_ghost \
    --volume /path/to/mariadb-persistence:/bitnami \
    bitnami/mariadb:latest
  ```

  *Note:* You need to give the container a name in order to Ghost to resolve the host

3. Create the Ghost container with host volumes:

  ```bash
  $ docker run -d --name ghost -p 80:2368 \
    -e ALLOW_EMPTY_PASSWORD=yes \
    -e GHOST_DATABASE_USER=bn_ghost \
    -e GHOST_DATABASE_NAME=bitnami_ghost \
    -e GHOST_HOST=localhost \
    --net ghost-tier \
    --volume /path/to/ghost-persistence:/bitnami \
    bitnami/ghost:latest
  ```

# Upgrade this application

Bitnami provides up-to-date versions of MariaDB and Ghost, including security patches, soon after they are made upstream. We recommend that you follow these steps to upgrade your container. We will cover here the upgrade of the Ghost container. For the MariaDB upgrade see https://github.com/bitnami/bitnami-docker-mariadb/blob/master/README.md#upgrade-this-image

1. Get the updated images:

  ```bash
  $ docker pull bitnami/ghost:latest
  ```

2. Stop your container

 * For docker-compose: `$ docker-compose stop ghost`
 * For manual execution: `$ docker stop ghost`

3. Take a snapshot of the application state

```bash
$ rsync -a /path/to/ghost-persistence /path/to/ghost-persistence.bkp.$(date +%Y%m%d-%H.%M.%S)
```

Additionally, [snapshot the MariaDB data](https://github.com/bitnami/bitnami-docker-mariadb#step-2-stop-and-backup-the-currently-running-container)

You can use these snapshots to restore the application state should the upgrade fail.

4. Remove the currently running container

 * For docker-compose: `$ docker-compose rm ghost`
 * For manual execution: `$ docker rm ghost`

5. Run the new image

 * For docker-compose: `$ docker-compose up ghost`
 * For manual execution ([mount](#mount-persistent-folders-manually) the directories if needed): `docker run --name ghost bitnami/ghost:latest`

# Configuration

## Environment variables

When you start the ghost image, you can adjust the configuration of the instance by passing one or more environment variables either on the docker-compose file or on the docker run command line. If you want to add a new environment variable:

 * For docker-compose add the variable name and value under the application section:

```yaml
ghost:
  image: bitnami/ghost:latest
  ports:
    - 80:2368
  environment:
    - GHOST_HOST=my_host
```

 * For manual execution add a `-e` option with each variable and value:

```bash
 $ docker run -d -p 80:2368 --name ghost --network=ghost-tier \
    -e GHOST_PASSWORD=my_password \
    -v /your/local/path/bitnami/ghost:/bitnami \
    bitnami/ghost
```

Available variables:

##### User and Site configuration
- `GHOST_HOST`: Hostname for Ghost.
- `GHOST_PORT_NUMBER`: Port number used in the generated application URLs. Default: **80**
- `GHOST_PROTOCOL`: Protocol to use in the application URLs. Valid values are "http" and "https". Default: **http**
- `GHOST_USERNAME`: Ghost application username. Default: **user**
- `GHOST_PASSWORD`: Ghost application password. Minimum length is 10 characters. Default: **bitnami123**
- `GHOST_EMAIL`: Ghost application email. Default: **user@example.com**
- `BLOG_TITLE`: Ghost blog title. Default: **User's Blog**

##### Use an existing database

- `MARIADB_HOST`: Hostname for MariaDB server. Default: **mariadb**
- `MARIADB_PORT_NUMBER`: Port used by MariaDB server. Default: **3306**
- `GHOST_DATABASE_NAME`: Database name that Ghost will use to connect with the database. Default: **bitnami_ghost**
- `GHOST_DATABASE_USER`: Database user that Ghost will use to connect with the database. Default: **bn_ghost**
- `GHOST_DATABASE_PASSWORD`: Database password that Ghost will use to connect with the database. No defaults.
- `ALLOW_EMPTY_PASSWORD`: It can be used to allow blank passwords. Default: **no**

##### Create a database for Ghost using mysql-client

- `MARIADB_HOST`: Hostname for MariaDB server. Default: **mariadb**
- `MARIADB_PORT_NUMBER`: Port used by MariaDB server. Default: **3306**
- `MARIADB_ROOT_USER`: Database admin user. Default: **root**
- `MARIADB_ROOT_PASSWORD`: Database password for the `MARIADB_ROOT_USER` user. No defaults.
- `MYSQL_CLIENT_CREATE_DATABASE_NAME`: New database to be created by the mysql client module. No defaults.
- `MYSQL_CLIENT_CREATE_DATABASE_USER`: New database user to be created by the mysql client module. No defaults.
- `MYSQL_CLIENT_CREATE_DATABASE_PASSWORD`: Database password for the `MYSQL_CLIENT_CREATE_DATABASE_USER` user. No defaults.
- `ALLOW_EMPTY_PASSWORD`: It can be used to allow blank passwords. Default: **no**

##### SMTP Configuration

To configure Ghost to send email using SMTP you can set the following environment variables:

 - `SMTP_HOST`: SMTP host.
 - `SMTP_PORT`: SMTP port.
 - `SMTP_USER`: SMTP account user.
 - `SMTP_PASSWORD`: SMTP account password.
 - `SMTP_FROM_ADDRESS`: SMTP from address.
 - `SMTP_SERVICE`: SMTP service to use.

This would be an example of SMTP configuration using a GMail account:

 * docker-compose:

```yaml
  ghost:
    image: bitnami/ghost:latest
    ports:
      - 80:2368
    environment:
      - GHOST_HOST=localhost
      - ALLOW_EMPTY_PASSWORD=yes
      - GHOST_DATABASE_USER=bn_ghost
      - GHOST_DATABASE_NAME=bitnami_ghost
      - SMTP_HOST=smtp.gmail.com
      - SMTP_USER=your_email@gmail.com
      - SMTP_PASSWORD=your_password
      - SMTP_FROM_ADDRESS="'Custom Name' <myemail@address.com>"
      - SMTP_SERVICE=GMail
```

 * For manual execution:

```bash
 $ docker run -d -p 80:2368 --name ghost --network=ghost-tier \
    -e GHOST_HOST=localhost \
    -e ALLOW_EMPTY_PASSWORD=yes \
    -e GHOST_DATABASE_USER=bn_ghost \
    -e GHOST_DATABASE_NAME=bitnami_ghost \
    -e SMTP_HOST=smtp.gmail.com \
    -e SMTP_SERVICE=GMail \
    -e SMTP_USER=your_email@gmail.com \
    -e SMTP_PASSWORD=your_password \
    -v /your/local/path/bitnami/ghost:/bitnami \
    bitnami/ghost
```

# Notable Changes

## 0.11.10-r2

- The ghost container has been migrated to a non-root container approach. Previously the container run as `root` user and the ghost daemon was started as `ghost` user. From now own, both the container and the ghost daemon run as user `1001`.
  As a consequence, the configuration files are writable by the user running the ghost process.

# Contributing

We'd love for you to contribute to this container. You can request new features by creating an [issue](https://github.com/bitnami/bitnami-docker-ghost/issues), or submit a [pull request](https://github.com/bitnami/bitnami-docker-ghost/pulls) with your contribution.

# Issues

If you encountered a problem running this container, you can file an [issue](https://github.com/bitnami/bitnami-docker-ghost/issues). For us to provide better support, be sure to include the following information in your issue:

- Host OS and version
- Docker version (`docker version`)
- Output of `docker info`
- Version of this container (`echo $BITNAMI_IMAGE_VERSION` inside the container)
- The command you used to run the container, and any relevant output you saw (masking any sensitive information)

# License

Copyright 2016-2019 Bitnami

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
