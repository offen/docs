---
layout: default
title: "Configuring and Deploying Offen on Heroku"
nav_order: 3
description: "A step by step tutorial on how deploy Offen on Heroku."
permalink: /running-offen/tutorials/configuring-deploying-offen-heroku/
parent: Tutorials
grand_parent: Running Offen
---

# Configuring and Deploying Offen on Heroku

This tutorial walks you through the steps needed to setup and deploy a single-node Offen instance on [Heroku][heroku] using PostgreSQL for storing data.

[heroku]: https://www.heroku.com/

---

## 1. Prerequistes

To follow the steps in this tutorial you will need to have created an account with Heroku and have the [Heroku CLI tool][heroku-cli] installed which is available for Linux, Windows and MacOS.

All resources created in this tutorial are free of charge. You might want to upgrade some of them to another plan with costs when running Offen in productio though.

---

__Heads Up__
{: .label .label-red }

If you do not want to use the command line, it's possible to do all of the following steps in the Heroku dashboard in your browser as well.

[heroku-cli]: https://devcenter.heroku.com/articles/heroku-cli

---

## 2. Create the Heroku app and attach a database

The first thing you will need to do is to create a new application:

```
$ heroku apps:create --region eu
Creating app... done, ⬢ glacial-sierra-90893, region is eu
https://glacial-sierra-90893.herokuapp.com/ | https://git.heroku.com/glacial-sierra-90893.git
```

In this example, Heroku created an application called `glacial-sierra-90893` for us, but you can also use a custom name for your application if you want to.

Next, provision a PostgreSQL database for your application:

```
$ heroku addons:create heroku-postgresql:hobby-dev --app glacial-sierra-90893       
Creating heroku-postgresql:hobby-dev on ⬢ glacial-sierra-90893... free
Database has been created and is available
 ! This database is empty. If upgrading, you can transfer
 ! data from another database with pg:copy
Created postgresql-rectangular-03979 as DATABASE_URL
Use heroku addons:docs heroku-postgresql to view documentation
```

---

## 3. Set the Configuration Values

### 3.1. Configure the Database

Offen needs to know which database dialect you are planning to use (`postgres`) and how to connect to it. The connection string of the database you just created can be found in the application's config values:

```
$ heroku config --app glacial-sierra-90893
=== glacial-sierra-90893 Config Vars
DATABASE_URL: postgres://xxxxxxxxxxxxxx:yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy@ec2-54-246-98-119.eu-west-1.compute.amazonaws.com:5432/zzzzzzzzzzzzzz
```

Copy this value and configure Offen for using the database:

```
heroku config:set \
  OFFEN_DATABASE_DIALECT=postgres \
  OFFEN_DATABASE_CONNECTIONSTRING=postgres://xxxxxxxxxxxxxx:yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy@ec2-54-246-98-119.eu-west-1.compute.amazonaws.com:5432/zzzzzzzzzzzzzz \
  --app glacial-sierra-90893
```

### 3.2. Create Secrets

Offen requires two unique **Base64 encoded secrets** for each instance to be configured. These values make sure you can login to your account safely. It is important that these values are unique to your instance, so please do not copy them from a webpage or similar.

On Linux or MacOS you can create them like this:

```
$ head -c 16 /dev/urandom | base64
Ar8BfwVi3vhqo3lvh0iizA== # do not use this secret
```

Alternatively you can use the `offen` command itself:

```
$ docker run --rm offen/offen:latest secret
time="2020-01-28T18:20:46Z" level=info msg="Created 16 bytes secret" secret="JKaK+adPexEUjgGlTzZbow==" # do not use this secret
```

Generate two values and configure the `OFFEN_SECRETS_EMAILSALT` and `OFFEN_SECRETS_COOKIEEXCHANGE` value:

```
heroku config:set \
  OFFEN_SECRETS_COOKIEEXCHANGE=Ar8BfwVi3vhqo3lvh0iizA== \
  OFFEN_SECRETS_EMAILSALT=YFI/pKJCxzrc23J4bfbJUQ== \
  --app glacial-sierra-90893
```

---

__Heads Up__
{: .label .label-red }

The value for `OFFEN_SECRETS_EMAILSALT` is not expected to ever change during the lifetime of a deployment. If this value gets lost or changes, you will not be able to log in again.

---

### 3.3. Configure Email

Offen needs to send transactional email for the following features:

- Inviting a new user to an account
- Resetting your password in case you forgot it

To enable this, you can add SMTP credentials, namely __Host, User, Password and Port__ to the `offen.env` file:

```
heroku config:set \
  OFFEN_SMTP_HOST="smtp.mysite.com" \
  OFFEN_SMTP_USER="me" \
  OFFEN_SMTP_PASSWORD="my-password" \
  OFFEN_SMTP_PORT="587" \
  --app glacial-sierra-90893
```

---

__Heads Up__
{: .label .label-red }

Offen will run without these values being set and try to fall back to a local `sendmail` install, yet please be aware that if you rely on any of the above features email delivery will be __very unreliable if not configured correctly__. You can always add this at a later time though.

---

### 3.4. Configure Offen to use the Correct Port

Heroku mandates the usage of `PORT` which is why you need to configure Offen to pick this up correctly:

```
heroku config:set OFFEN_SERVER_USENAKEDPORT=1 --app glacial-sierra-90893
```

## 4. Deploying the Application

Now that all of the required configuration values are set, we are ready to deploy the application using our `offen/offen` Docker image using the Heroku container stack.

### 4.1. Setting the Stack

Tell your app to use the `container` stack:

```
heroku stack:set container --app glacial-sierra-90893
```

### 4.2. Create the Dockerfile and heroku.yml

The `container` stack requires a Dockerfile for defining the application and a `heroku.yml` for defining your application structure.

In an empty directory, create a `Dockerfile` looking like this:

```
FROM offen/offen:latest
CMD ["serve"]
```

---

__Heads Up__
{: .label .label-red }

There is an issue in the last stable release of Offen that keeps it from working correctly in Heroku, so this tutorial is using the `latest` channel. If you consider using this setup in production, please lock the revision you are deploying.

---

`heroku.yml` is a simple file mapping the `web` process to the above Dockerfile:

```yml
build:
  docker:
    web: Dockerfile
```

### 4.3. Push the Setup to Heroku

To deploy your setup create a Git repository in your directory and push it to Heroku using the Git URL shown when creating the application:

```
git init
git remote add heroku https://git.heroku.com/glacial-sierra-90893.git
git add -A
git commit -m 'Add Offen'
git push heroku master
```

You should now see Heroku building the Docker image and deploying Offen.

---

## 5. Running the `setup` Command

The final step for your installation is now to create an account that you can use to collect usage data and log in. To do so, run the `setup` command on your newly created deployment:

```
heroku run --app glacial-sierra-90893 -- setup -email="me@mysite.com" -name="my-site" -stdin-password
```

When finished, the command has created an account for you, using the given name and credentials.

### Test the Setup

You can now head to the running application at `https://glacial-sierra-90893.herokuapp.com/login` and login using your given credentials.

---

## 6. Setting up a Custom Domain

In a real world setup, you will likely want to make Offen available as a subdomain of your own domain. To do so, configure Heroku to use your desired custom domain:

```
$ heroku domains:add offen.mysite.com --app glacial-sierra-90893
Configure your app's DNS provider to point to the DNS Target comparative-feijoa-ruly5q3ppq66ettk7jdi8rp4.herokudns.com.
  For help, see https://devcenter.heroku.com/articles/custom-domains

The domain heroku.offen.dev has been enqueued for addition
Run heroku domains:wait 'offen.mysite.com' to wait for completion
Adding heroku.offen.dev to ⬢ glacial-sierra-90893... done
```

Now, you can set a CNAME record from your desired domain to the target given in the response.