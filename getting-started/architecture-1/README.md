---
hidden: true
noIndex: true
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

# Architecture

Oktopus Controller backend is made up of some microservices written in [Go](https://go.dev/) because of the language versatility, speed, and small resource usage footprint. All the microservices communicate with each other through [NATS](https://nats.io/), which is a robust message system, really fast, built for IoT and edge applications.   The database picked is [MongoDB](https://www.mongodb.com/), which makes possible to have flexible data, saved in the right manner based on our needs.&#x20;

The frontend is made with the most modern frameworks and design system available in the market. It has the [Material UI ](https://mui.com/material-ui/getting-started/)components lib integrated with [NextJS](https://nextjs.org/), which ensures a smooth single page app experience.&#x20;

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>High-Level Components</p></figcaption></figure>

All the different protocols used to manage the devices are abstracted to NATS message system, this allow us to have different components/services deployed in different servers, and locations, as long as the connection to NATS cluster is maintained.

Software components are deployed within [Docker](https://www.docker.com/) containers. The most straigthforward way to install Oktopus Controller is using docker compose. For high availability production environments we suggest to use Kubernetes.

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>Microservices Overview</p></figcaption></figure>

