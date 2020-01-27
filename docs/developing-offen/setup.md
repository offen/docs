---
layout: page
title: Setup
nav_order: 2
description: "How to setup the development environment for Offen."
permalink: /developing-offen/setup/
parent: Developing Offen
---

# Setup

## First run

After cloning the [repository][], you will need to do two things:

1. Pull the required Docker images and install each container's dependencies:
  ```
  make setup
  ```
1. Bootstrap the development database:
  ```
  make bootstrap
  ```
You should now be able to start your local version of offen using:
```
make up
```

To see if the application is working as intended visit <http://localhost:8080> in your browser or use the application's health check:

```
➜  ~ curl -X GET http://localhost:8080/healthz
{"ok":true}
```

You can stop the application using `Ctrl+C`, in case you want to tear down the current environment and delete the data stored in the local database you can use:

```
make down
```

[repository]: https://github.com/offen/offen

## Updating dependencies

In case the dependencies consumed by one of the containers have changed, you can run the following command to get an up-to-date version:

```
make update
```

## Rebuilding Docker images

In case the Dockerfiles defining the development images have changes you will need to rebuild the local images using:

```
make dev-build
```

## Running database migrations

If your local database schema is out of date and you need to apply the latest migrations, run

```
make migrate
```
