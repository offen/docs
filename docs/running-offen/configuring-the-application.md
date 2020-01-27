---
layout: default
title: Configuring The Application
nav_order: 3
description: "How to configure an Offen instance at runtime."
permalink: /running-offen/configuring-the-application/
parent: Running Offen
---

# Configuring the Application at Runtime

The main interface to configuring Offen at runtime is using your systems environment variables. All variables are following the pattern of `OFFEN_<scope>_<key>`, e.g. `OFFEN_SERVER_PORT`. In addition to setting variables in the environment Offen also allows for setting these values in [`env` files][dotenv].

[dotenv]: https://github.com/joho/godotenv

## Lookup order

### On Linux and MacOS

In case the `-envfile` flag was supplied with a value when invoking a command, Offen will use this file. In case no such flag was given, Offen looks for files named `offen.env` in the following locations:

- In the current working directory
- In `~/.config`
- In `$XDG_CONFIG_HOME`
- In `/etc/offen`

### On Windows

In case the `-envfile` flag was supplied with a value when invoking a command, Offen will use this file. In case no such flag was given, Offen expects a file named `offen.env` to be present in the current working directory.

## Configuration format

`env` files will specify the same keys the environment variables use, e.g.:

```
OFFEN_SERVER_PORT = "4000"
OFFEN_DATABASE_DIALECT = "sqlite3"
OFFEN_DATABASE_CONNECTIONSTRING = "/opt/offen/data/db.sqlite"
```

## Configuration options

The `SERVER` namespace collects settings that affect the behavior of the HTTP server that is serving the application.

### `OFFEN_SERVER_PORT`

Defaults to `8080`.

The port the application listens on.

### `OFFEN_SERVER_REVERSEPROXY`

Defaults to `false`.

If set to `true` the application will assume it is running behind a reverse proxy. This means it does not add caching or security related headers to any response. Logging requests to `stdout` is also disabled.

### `OFFEN_SERVER_SSLCERTIFICATE`

In case you own a SSL certificate that is valid for the domain you are planning to serve your Offen instance from, you can pass the location of the certificate file using this variable. It also requires `OFFEN_SERVER_SSLKEY` to be set.

### `OFFEN_SERVER_SSLKEY`

In case you own a SSL certificate that is valid for the domain you are planning to serve your Offen instance from, you can pass the location of the key file using this variable. It also requires `OFFEN_SERVER_SSLCERTIFICATE` to be set.

### `OFFEN_SERVER_AUTOTLS`

In case you want Offen to automatically request a free SSL certifcate from LetsEncrypt you can use this parameter and assign the domain name (e.g. `offen.mydomain.org`) you are planning to serve Offen from. This will have the application automatically handle certificate issuing and renewal.

### `OFFEN_SERVER_CERTFICATECACHE`

Defaults to `/var/www/.cache`

When using the AutoTLS feature, this sets the location where Offen will be caching certificates.

---

The `DATABASE` namespace collects settings regarding the connected persistence layer. If you do not configure any of these, Offen will be able to start, but data will not persist as it will be saved into a local temporary database.

### `OFFEN_DATABASE_DIALECT`

Defaults to `sqlite3`.

The SQL dialect to use. Supported options are `sqlite3`, `postgres` or `mysql`.

### `OFFEN_DATABASE_CONNECTIONSTRING`

Defaults to `/tmp/offen.db`

The connection string or location of the database. For `sqlite3` this will be the location of the database file, for other dialects, it will be the URL the database is located at, including credentials needed to access it.

__Heads up:__ Using the default value will mean all data is lost at the next system restart, so you will likely not want to use the default for purposes other than testing.

---

The `APP` namespace affects how the application will behave.

### `OFFEN_APP_DEVELOPMENT`

Defaults to `false`.

If set to `true` the application assumes it is running in development mode, which will add verbose logging and disable production specific optimizations.

### `OFFEN_APP_EVENTRETENTIONPERIOD`

Defaults to `4464h`.

This value specifies how long events are kept before being deleted. It also determines the expiry date of the user cookie that is used to enable users to access their data. The value is expected to be specified as a valid [Golang duration string][go-duration]. Valid time units are "ns", "us" (or "µs"), "ms", "s", "m", "h".

[go-duration]: https://golang.org/pkg/time/#ParseDuration


### `OFFEN_APP_LOCALE`

Defaults to `en`.

The locale the application will use when displaying user facing text. Right now, only `en` is supported. In case you want to contribute to Offen by adding a new locale, we'd love to hear from you.

### `OFFEN_APP_LOGLEVEL`

Defaults to `info`.

Specifies the application's log level. Possible values are `debug`, `info`, `warn`, `error`. If you use a level higher than `info`, access logging - which is happening at `info` level - will be suppressed.

### `OFFEN_APP_SINGLENODE`

Defaults to `true`.

In case you want to run Offen as a horizontally scaling service, you can set this value to `true`. This will disable all cron jobs and similar that handle automated database migration and event expiration.

---

The `SECRETS` namespace collects secrets used for signing cookies and hashing account data. All of the values are required to be populated.

### `OFFEN_SECRETS_COOKIEEXCHANGE`

No default value. __Required__.

A Base64 encoded secret that is used for signing cookies. Ideally, it is of 16 bytes length. Changing this value during the lifecycle of the application means all currently active account user sessions are invalidated and users would have to log in again.

### `OFFEN_SECRETS_EMAILSALT`

No default value. __Required__.

Offen does not store Email addresses in plain text, but in a hashed form. This salt is used for hashing addresses. Ideally, it is of 16 bytes length. This value is not expected to change during the lifecycle of an Offen instance. Changing it would mean all existing user accounts **cannot log in anymore**.

---

`SMTP` is a namespace used for configuring how transactional email is being sent. If any of these values is missing, Offen will fallback to using local `sendmail` which will likely be unreliable, so configuring these values is recommended.

### `OFFEN_SMTP_USER`

No default value.

The SMTP user name used when sending transactional email.

### `OFFEN_SMTP_PASSWORD`

No default value.

The SMTP user name used when sending transactional email.

### `OFFEN_SMTP_HOST`

No default value.

The SMTP hostname used when sending transactional email.

### `OFFEN_SMTP_PORT`

Default value `587`.

The SMTP port used when sending transactional email.
