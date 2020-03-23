# Introduction

This will build a container for [LemonLDAP::NG](https://lemonldap-ng.org/) an elegant web based manager for Authentication (SAML, OpenID Connect, CAS) using Nginx

* This Container uses a [customized Alpine Linux base](https://hub.docker.com/r/naviat/alpine) which includes [s6 overlay](https://github.com/just-containers/s6-overlay) enabled for PID 1 Init capabilities, [zabbix-agent](https://zabbix.org) for individual container monitoring, Cron also installed along with other tools (bash,curl, less, logrotate, nano, vim) for easier management. It also supports sending to external SMTP servers..

* Sane Defaults to have a working solution by just running the image
* Automatically generates configuration files on startup, or option to use your own
* Option to just use image as a Handler for external servers
* Handler Option to use file base socket or listen on TCP
* Fail2ban Included for blocking brute force attacks.
* Ready to work out the box for SAML, OpenID, 2FA/2OTP
* Additional modules compiled for Redis, Mysql, Postgres, LDAP Session/Config Storage
* Choice of Logging (Console, File, Syslog)
* Used in Production for an educational institution serving over 10000 users.

*This is an incredibly complex piece of software that will tries to get you up and running with sane defaults, you will need to switch eventually over to manually configuring the configuration file when depending on your usage case*


# Table of Contents

- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [Configuration](#configuration)
  - [Database](#database)
  - [Data Volumes](#data-volumes)
  - [Environment Variables](#environmentvariables)
  - [Networking](#networking)
- [Maintenance](#maintenance)
  - [Shell Access](#shell-access)
- [References](#references)

# Prerequisites

You must have access to create records on your DNS server to be able to setup the demo installation before configuration.


# Installation

Automated builds of the image are available on [Docker Hub](https://hub.docker.com/naviat/lemonldap) and is the 
recommended method of installation.


```bash
docker pull naviat/lemonldap:<tag>
```

The following image tags are available:

* `latest` - LemonLDAP 2.0.x Branch w/ Alpine Linux

# Quick Start


* Add records for your main, and manager names into DNS (ie `handler.sso.example.com`. `manager.sso.example.com`, `sso.example.com`, `test.sso.example.com`)

* Set various [environment variables](#environment-variables) to understand the capabilities of this image. A Sample `docker-compose.yml` is provided that will work right out of the box for most people without any fancy optimizations.

* Map [persistent storage](#data-volumes) for access to configuration and data files for backup.

Once run, visit the Manager URL and login as `haidv/haidv`

# Configuration

### Persistent Storage

The following directories should be mapped for persistent storage in order to utilize the container effectively.

| Folder    | Description |
|-----------|-------------|
| `/etc/lemonldap-ng/` | (Optional) - LemonLDAP core configuration files. Auto Generates on Container startup |
| `/var/lib/lemonldap-ng/conf` | Actual Configuration of LemonLDAP (lmConf-X.js files) |
| `/var/lib/lemonldap-ng/sessions` | (Optional) - Storage of Sessions of users
| `/var/lib/lemonldap-ng/psessions` | (Optional) - Storage of Sessions of users
| `/assets/custom` | Ability to overwrite themes/inject into image upon bootup for theming /etc.
| `/www/logs` | Log files for individual services

### Environment Variables

Along with the Environment Variables from the [Base image](https://hub.docker.com/r/naviat/alpine) and [Nginx Image](https://hub.docker.com/r/naviat/nginx),  below is the complete list of available options that can be used to customize your installation.

There are a huge amount of configuration variables and it is recommended that you get comfortable for a few hours with the [LemonLDAP::NG Documentation](https://lemonldap-ng.org/documentation/2.0/start)

You will eventually based on your usage case switch over to `SETUP_TYPE=manual` and edit your own `lemonldap-ng.ini`. While I've tried to make this as easy to use as possible, once in production you'll find much better success with large implementations with this approach.

By Default this image is ready to run out of the box, without having to alter any of the settings with the exception of the `_HOSTNAME` vars. You can also change the majority of these settings from within the Manager. There are instances where these variables would want to be set if you are running multiple handlers or need to enforce a Global Setting for one specific installation.

| Parameter | Description |
|-----------|-------------|
| `SETUP_TYPE` | Default: `AUTO` to auto generate lemonldap-ng.ini on bootup, otherwise let admin control configuration. |
| `MODE` | Type of Install - `HANDLER` for handler duties only, Default `MASTER` for Portal, Manager, Handler |
| `CONFIG_TYPE` | Configuration type (`FILE`, `SOAP`, `REST`) - Default `FILE` |
| `DOMAIN_NAME` | Your domain name e.g. `example.org` |
| `MANAGER_HOSTNAME` | FQDN for Manager e.g. `manager.sso.example.org` |
| `PORTAL_HOSTNAME` | FQDN for public portal/main URL e.g. `sso.example.org` |
| `HANDLER_HOSTNAME` | FQDN for Configuration reload URL e.g. `handler.sso.example.org` |
| `TEST_HOSTNAME` | FQDN for test URL to prove that LemonLDAP works e.g. `test.sso.example.org` |
| `LOG_TYPE` | How to Log - Options `CONSOLE, FILE, SYSLOG` - Default: `CONSOLE`  |
| `LOG_LEVEL` | LogLevel - Options `warn, notice, info, error, debug` Default `info` |
| `USER_LOG_TYPE` | How to Log User actions - Options `CONSOLE, FILE, SYSLOG` - Default: `CONSOLE`  |

#### SOAP Settings

Depending if `SOAP` was chosedn for `CONFIG_TYPE`, these variables would be used.

| Parameter | Description |
|-----------|-------------|
| `SOAP_HOST` | Hostname of Master SOAP Server e.g. `https://sso.example.com/index.pl/config` |
| `SOAP_USER` | Username to fetch Configuration Information |
| `SOAP_PASS` | Password to fetch Configuration Information |

#### REST Settings

Depending if `REST` was chosedn for `CONFIG_TYPE`, these variables would be used.

| Parameter | Description |
|-----------|-------------|
| `REST_HOST` | Hostname of Master REST Server e.g. `https://sso.example.com/index.psgi/config/` |
| `REST_USER` | Username to fetch Configuration Information |
| `REST_PASS` | Password to fetch Configuration Information |

### Portal Settings
| Parameter | Description |
|-----------|-------------|
| `PORTAL_CACHE_TYPE` | Only Cache Type available for now - Default `FILE` |
| `PORTAL_LOCAL_CONF` | See LemonLDAP Documentation Default: `FALSE` |
| `PORTAL_SKIN` | Default: `bootstrap` - See LemonLDAP Documentation |
| `PORTAL_TEMPLATE_DIR` | Default: `/usr/share/lemonldap-ng/portal/templates` |
| `PORTAL_LOG_TYPE` | Override Portal Log - Options `CONSOLE, FILE, SYSLOG` - Default: `CONSOLE`  |
| `PORTAL_LOG_LEVEL` | Override Portal LogLevel - Options `warn, notice, info, error, debug` Default `info` |
| `PORTAL_USER_LOG_TYPE` | Override Portal Log User actions - Options `CONSOLE, FILE, SYSLOG` - Default: `CONSOLE`  |
| `PORTAL_ENABLE_REST` | Allow REST access to the Portal - Default `FALSE` |
| `PORTAL_ENABLE_SOAP` | Allow SOAP access to the Portal - Default `FALSE` |
| `PORTAL_RESTSOAP_ALLOWED_IPs` | If above options enabled, provide comma seperated list of IP/Network to allow access Default `0.0.0.0/0` |
| `ENABLE_IMPERSONATION` | If you wish to allow impersonation using a seperate theme set to `TRUE` - Default: `FALSE` |
| `IMPERSONATE_HOSTNAME` | Hostname to use to load the custom impersonation theme |
| `IMPERSONATE_THEME` | Theme to use to load the impersonation theme |

* With impersonation, if you enable it, it will add a new field to your login screen, which may not be what you want if this is a production system. You will need to create two custom themes (one as a replica of bootstrap, and one for impersonation). In the custom theme, make modifications to `login.tpl` to stop it from loading impersonation.tpl, yet in your impersonation theme, leave it in there. Then, when one of your admin/support team visits the custom `IMPERSONATE_HOSTNAME` you have defined it will load the full theme with allows to impersonate, where as the default theme will not show this.*


### Handler Settings
| Parameter | Description |
|-----------|-------------|
| `CACHE_TYPE` | Session Cache type (`FILE` only available for now) - Default `FILE` |
| `CACHE_TYPE_FILE_NAMESPACE` | Default: `lemonldap-ng-config` |
| `CACHE_TYPE_FILE_EXPIRY` | Default: `600` |
| `CACHE_TYPE_FILE_DIR_MASK` | Default: `007` |
| `CACHE_TYPE_FILE_PATH` | Default: `/tmp` |
| `CACHE_TYPE_FILE_DEPTH` | Default: `0` |
| `HANDLER_ALLOWED_IPS` | If you need to access access to `/reload` other than localhost add a comma seperated list or hosts or networks here e.g. `172.16.0.0/12,192.168.0.253` |
| `HANDLER_CACHE_TYPE` | Default: `FILE` |
| `HANDLER_CACHE_TYPE_FILE_NAMESPACE` | Default: `lemonldap-ng-sessions` |
| `HANDLER_CACHE_TYPE_FILE_EXPIRY` | Default: `600` |
| `HANDLER_CACHE_TYPE_FILE_DIR_MASK` | Default: `007` |
| `HANDLER_CACHE_TYPE_FILE_PATH` | Default: `/tmp` |
| `HANDLER_CACHE_TYPE_FILE_DEPTH` | Default: `3` |
| `HANDLER_SOCKET_TCP_ENABLE` | Enable TCP Connections to socket instead of /var/run/llng-fastcgi-server/llng-fastcgi.sock - Default `TRUE` |
| `HANDLER_SOCKET_TCP_PORT` | Port to listen on for Handler Default `2884` |
| `HANDLER_STATUS` | Allow Status on Handler Default: `TRUE` |
| `HANDLER_REDIRECT_ON_ERROR` | Default: `TRUE` |
| `HANDLER_LOG_TYPE` | Override Handler Log - Options `CONSOLE, FILE, SYSLOG` - Default: `CONSOLE`  |
| `HANDLER_LOG_LEVEL` | Override Handler LogLevel - Options `warn, notice, info, error, debug` Default `info` |
| `HANDLER_PROCESSES` | Amount of LLNG Handler processes to spawn - Default: `7` |
| `HANDLER_USER_LOG_TYPE` | Override Handler Log User actions - Options `CONSOLE, FILE, SYSLOG` - Default: `CONSOLE`  |

### Manager Options

| Parameter | Description |
|-----------|-------------|
| `MANAGER_PROTECTION` | Default: `manager` |
| `MANAGER_LOG_LEVEL` | Default: `warn` |
| `MANAGER_STATIC_PREFIX` | Default: `/static` |
| `MANAGER_TEMPLATE_DIR` | Default: `/usr/share/lemonldap-ng/manager/templates` |
| `MANAGER_LANGUAGE` | Default: `en` |
| `MANAGER_ENABLED_MODULES` | Default: `"conf, sessions, notifications"` |
| `MANAGER_LOG_TYPE` | Override Manager Log - Options `CONSOLE, FILE, SYSLOG` - Default: `CONSOLE`  |
| `MANAGER_LOG_LEVEL` | Override Manager LogLevel - Options `warn, notice, info, error, debug` Default `info` |
| `MANAGER_USER_LOG_TYPE` | Override Manager Log User actions - Options `CONSOLE, FILE, SYSLOG` - Default: `CONSOLE`  |
      

### Networking

The following ports are exposed.

| Port      | Description |
|-----------|-------------|
| `80` | HTTP |
| `2884` | LemonLDAP Handler |

# Maintenance
#### Shell Access

For debugging and maintenance purposes you may want access the containers shell. 

```bash
docker exec -it (whatever your container name is e.g. lemonldap) bash
```

# References

* https://lemonldap-ng.org
