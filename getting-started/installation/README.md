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

# Installation

Before you dive into the installation methods and documentation about, make sure you've read the [requirements section](../requirements/).

> The following tutorials don't cover all possible configurations and deployment modes of Oktopus Controller, many adjusts as TLS encryption of MTPs and NATS are not documented yet, although the software already supports it. Your help is welcome to help us cover this topics, and soon we hope to be launching content related to encryption, USP and CWMP best practices and different possibilites to install Oktopus as in a cloud provider, on-premises, or hybrid.

## Quick Start

If you're a developer or someone who just wants to try Oktopus and get a better understanding of CWMP and/or USP standards, don't waste time. Have the software running with one command:

```
wget https://github.com/OktopUSP/oktopus/archive/refs/heads/main.zip && unzip main && cd oktopus-main/deploy/compose && COMPOSE_PROFILES=nats,controller,cwmp,mqtt,stomp,ws,adapter,frontend docker compose up -d
```

The web interface will become accessible at port 80:

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>Oktopus web first acces</p></figcaption></figure>
