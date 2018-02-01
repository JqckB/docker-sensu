Dockerized Sensu
================

This git repo provides [Sensu](https://sensuapp.org/) in a Docker container.

Project: [https://github.com/sstarcher/docker-sensu]
(https://github.com/sstarcher/docker-sensu)

Docker image: [https://registry.hub.docker.com/u/sstarcher/sensu/]
(https://registry.hub.docker.com/u/sstarcher/sensu/)

[![](https://images.microbadger.com/badges/image/sstarcher/sensu.svg)](http://microbadger.com/images/sstarcher/sensu "Get your own image badge on microbadger.com")
[![Docker Registry](https://img.shields.io/docker/pulls/sstarcher/sensu.svg)](https://registry.hub.docker.com/u/sstarcher/sensu)&nbsp;

This is a base container for Sensu Core. It contains `sensu-api`, `sensu-client`, `sensu-server`, but does *not* contain any plugins.

Default configuration is to use `redis` as the transport.  This allows us to not need `rabbitmq`.

This container can be configured to use runtime system information for checks and metrics from it's *host*.


## Note

Installed plugins for now can change without warning. If you need a specific plugin installed either build a container based off of this one our use `RUNTIME_INSTALL` to ensure your plugins are installed.


## Configuration

The client by default only loads files from `/etc/sensu/conf.d`.

The API by default loads files from:

- `/etc/sensu/conf.d`
- `/etc/sensu/check.d`

The server by default loads files from:

- `/etc/sensu/conf.d`
- `/etc/sensu/check.d`
- `/etc/sensu/handlers`

These defaults could be configured via environment variables:

```
ENV CONFIG_DIR /etc/sensu/conf.d
ENV CHECK_DIR /etc/sensu/check.d
ENV HANDLERS_DIR /etc/sensu/handlers
```

If you want `sensu-client` to use runtime system information for checks and metrics from container's *host* system (*not* from `sensu` container itself) you should:

1. Define volumes to access host's filesystem from `sensu` container :

  ```
  /dev:/host_dev/:ro
  /proc:/host_proc/:ro
  /sys:/host_sys/:ro
  ```

2. Redefine environment variables :

  ```
  HOST_DEV_DIR: /host_dev
  HOST_PROC_DIR: /host_proc
  HOST_SYS_DIR: /host_sys
  ```

All Sensu plugins will be automatically configured to use these paths instead of default ones.


Dependencies:
  - Server
    - rabbitmq - If transport is redis this is not used
    - redis
    - api
  - Api
    - rabbitmq - If transport is redis this is not used
    - redis
  - Client
    - transport - rabbitmq or redis


```
RABBITMQ_PORT 5672
RABBITMQ_HOST rabbitmq
RABBITMQ_USER guest
RABBITMQ_PASSWORD guest
RABBITMQ_VHOST /
RABBITMQ_SSL_SUPPORT false
RABBITMQ_SSL_CERT ""
RABBITMQ_SSL_KEY ""

REDIS_HOST redis
REDIS_PORT 6379
REDIS_PASSWORD ""
REDIS_DB 0
REDIS_AUTO_RECONNECT true
REDIS_RECONNECT_ON_ERROR false
```

Client specific settings.

```
CLIENT_NAME *no default*
CLIENT_ADDRESS *no default*
CLIENT_SUBSCRIPTIONS all, default
CLIENT_KEEPALIVE_HANDLER default
```

Settings required for Server and API.  Defaults to no username or password.
```
API_PORT 4567
API_BIND 0.0.0.0
API_HOST api
API_USER *no default*
API_PASSWORD *no default*
```

You can use `docker-compose.yaml` to run everything locally.

`RUNTIME_INSTALL` will allow you to install additional plugins from github during runtime.  The install format is USERNAME/repo@TAG.  The default USERNAME is sensu-plugins and the default TAG is master.  In place of a TAG a full git sha may be used.  

Examples:
* https://github.com/sensu-plugins/sensu-plugins-disk-checks
  * RUNTIME_INSTALL='disk-checks' will install the master branch
  * RUNTIME_INSTALL='sensu-plugins/disk-checks@2.5.1 will install tag 0.5.1
* https://github.com/sstarcher/sensu-plugins-http
  * RUNTIME_INSTALL='sstarcher/http` will install the master branch from the sstarcher organization on github for http

Multiple plugins will be installed in parallel. However, if fetching and compiling too many dependencies at the same time cause issues, set `PARALLEL_INSTALLATION` to 0.

If you want to keep the installed build tools and the apt cache, set `UNINSTALL_BUILD_TOOLS` to 0.

`GEM_SOURCES` can be used to add additional gem sources (such as https://ruby.taobao.org/ for China).
