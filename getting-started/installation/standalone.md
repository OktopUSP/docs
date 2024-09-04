---
layout:
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Standalone

The standalone mode installs all the software components into one server. It's pretty straighforward, but becomes a single point of failure. Anyway, it should be enough for small-medium sized workloads.

## Requirements

* docker
* docker compose
* wget
* unzip

Have the services ports available internally or public accordingly to Oktopus [network requirement section](../requirements/network.md). Keep in mind it's always advisable to have a firewall in front of your server and have controll of the traffic policies.

This tutorial was tested against OS Ubuntu 24.04, but it must work for other linux systems too.

## Installation

First of all, let's download Oktopus codebase:

```
wget https://github.com/OktopUSP/oktopus/archive/refs/heads/main.zip && unzip main
```

Then go into the deploy/compose folder:

```bash
cd oktopus-main/deploy/compose
```

Edit ".env.controller" file last line with a secret value, this key will be used to sign JWTs used in Oktopus users authentication process.

```
nano .env.controller
```

{% code title=".env.controller" %}
```
MONGO_URI=mongodb://mongo_usp:27017
NATS_URL=nats://oktopususer:oktopuspw@msg_broker:4222
NATS_ENABLE_TLS="true"
CLIENT_CRT=/tmp/nats/config/cert.pem
CLIENT_KEY=/tmp/nats/config/key.pem
SERVER_CA=/tmp/nats/config/rootCA.pem
SECRET_API_KEY=<super-secret-key>
```
{% endcode %}

Once you've done the steps above, still inside "oktopus-main/deploy/compose" folder, run:

```
COMPOSE_PROFILES=nats,controller,cwmp,mqtt,stomp,ws,adapter,frontend,portainer docker compose up -d
```

Congratulations! Oktopus Web interface will be available at port 80.
