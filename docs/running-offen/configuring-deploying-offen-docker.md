---
layout: default
title: "Tutorial: Configuring and Deploying Offen Using Docker"
nav_order: 5
description: "A step by step tutorial on how to use Docker to deploy Offen."
permalink: /running-offen/configuring-deploying-offen-docker/
parent: Running Offen
---

# Configuring and Deploying Offen Using Docker

This tutorial walks you through the steps needed to setup and deploy a standalone, single-node Offen instance that is using a local SQLite file as its database backend. Other and more complex configuration options are documented in ["Configuring The Application At Runtime"][config-docs]. General considerations for deploying Offen are described in ["Requirements for Installing Offen"][installation].

[config-docs]: /running-offen/configuring-the-application/
[installation]: /running-offen/installation-requirements/

## Pulling the Docker Image

The official Docker image is available as [`offen/offen`][docker-hub] on Docker Hub. The most recent official release is tagged as `stable`:

```sh
docker pull offen/offen:stable
```

If you are feeling adventurous or require features that are not yet available in a release you can also use the `latest` tag which represent the latest state of development, but might be unstable.

[docker-hub]: https://hub.docker.com/r/offen/offen

## Choosing a Location for Storing Your Data

In the simple setup described in this tutorial Offen needs to persist the following files:

- a database file
- a configuration file
- cache files the SSL certificates

This tutorial assumes you are on Linux, so we create a `~/offen` directory in which we will store our data. You can use basically any other location though, so feel free to change this.

In your `~/offen` directory, create a `cache` directory, which can be used for caching certificates:

```
~/offen
└── cache
```

## Creating a Configuration and Database File

Offen can source configuration from environment variables as well as configuration files. In this tutorial we will use a configuration file called `offen.env` and a database file called `offen.db` which you can create as empty files in your data directory:

```
~/offen
├── cache
└── offen.env
└── offen.db
```

## Setting the Database Location

Offen needs to know where the database file is located. To let it know, put the following line in your `offen.env` file:

```
OFFEN_DATABASE_CONNECTIONSTRING = "/root/offen.db"
```

## Running the `setup` Command

Now that we have defined the database location, Offen lets you setup a new instance using the `setup` command:

```
docker run -it --rm \
  -v /home/you/offen/cache:/var/www/.cache \
  --mount type=bind,src=/home/you/offen/offen.env,dst=/root/offen.env \
  --mount type=bind,src=/home/you/offen/offen.db,dst=/root/offen.db \
  offen/offen:stable setup \
  -email me@mysite.com \ # the email used for login
  -name mysite \ # your account name, this will not be displayed to users
  -stdin-password \ # this will prompt for you password
  -populate # this will automatically create required secrets for you
```

When finished, the command has created an account for you, using the given name and credentials.

Your `offen.env` file will now look something like this:

```
OFFEN_DATABASE_CONNECTIONSTRING="/root/offen.db"
OFFEN_SECRETS_COOKIEEXCHANGE="uNrZP7r5fY3sfS35tbzR9w==" # do not use this secret in production
OFFEN_SECRETS_EMAILSALT="uVXyuzCcpTim0v7uChCs1UA==" # do not use this secret in production
```

## Setting up AutoTLS

Offen requires a secure connection and can automatically acquire a renew SSL certificates from LetsEncrypt for your domain. All you need to do is add the domain you want to serve Offen from to your `offen.env` file:

```
OFFEN_SERVER_AUTOTLS=offen.mysite.com
```

To make sure the automatic certificate creation and renewal works, make sure your host system exposes __both port 80 and 443__ to the public internet.

## Setting up Email

Offen needs to send transactional email for the following use cases:

- Inviting a new user to an account
- Resetting your password in case you forgot it

To enable this, you can add SMTP credentials, namely __Host, User, Password and Port__ to the `offen.env` file:

```
OFFEN_SMTP_HOST="smtp.mysite.com"
OFFEN_SMTP_USER="me"
OFFEN_SMTP_PASSWORD="my-password"
OFFEN_SMTP_PORT="587"
```

Offen will run without these values being set and try to fall back to a local `sendmail` install, yet please be aware that if you rely on any of the above features email delivery will be __very unreliable if not configured correctly__.

## Verifying your config file

Your config file should now contain an entry for each of these values:

```
OFFEN_DATABASE_CONNECTIONSTRING="/root/offen.db"
OFFEN_SECRETS_COOKIEEXCHANGE="uNrZP7r5fY3sfS35tbzR9w==" # do not use this secret in production
OFFEN_SECRETS_EMAILSALT="VXyuzCcpTim0v7uChCs1UA==" # do not use this secret in production
OFFEN_SERVER_AUTOTLS=offen.mysite.com
OFFEN_SMTP_HOST="smtp.mysite.com"
OFFEN_SMTP_USER="me"
OFFEN_SMTP_PASSWORD="my-password"
OFFEN_SMTP_PORT="587"
```

If all of this is populated with the values you expect, you're ready to use Offen.

## Starting the Application

To start Offen use the main `offen` command:

```
docker run -d \
  -v /home/you/offen/cache:/var/www/.cache \
  --mount type=bind,src=/home/you/offen/offen.env,dst=/root/offen.env \
  --mount type=bind,src=/home/you/offen/offen.db,dst=/root/offen.db \
  offen/offen:stable
```

Once the application has started, you can use `docker ps` to check if it's up and running:

```
$ docker ps
CONTAINER ID        IMAGE                COMMAND                  CREATED             STATUS              PORTS                    NAMES
70653aca75b4        offen/offen:stable   "offen"                  5 minutes ago       Up 5 minutes        80/tcp, 443/tcp          nice_murdock
```

Your instance is now ready to use. Once you have setup DNS to point at your host system, you can head to `https://offen.mysite.com/login` and login to your account.