---
description: A brief explanation of the USP components and how it works.
---

# Architecture

## Controller

A USP controller manages a set of service elements represented in a USP agent’s data model. A USP controller database contains all agents within a deployment, capturing states and capabilities, and helps instantiate use case changes. The Controller is responsable to manage USP agents, interacting with the Agent Data Model executing CRUD operations and actions in the device.

## Agent

A USP Agent is responsible for exposing a set of "Service Elements" (essentially, a data model composed of objects and parameters that represent a specific set of functionality) for consumption by a Controller. While USP is capable of being used in many different environments, the home network is expected to be the most common environment, and in this environment a USP Agent would reside in a piece of Customer Premise Equipment (CPE), e.g. broadband home router, Wi-Fi access point, IoT gateway.

## Messages Encode

The messages exchanged betweeen a USP Agent and Controller might be encrypted with TLS. All the content is structured using [ProtoBuffers](https://protobuf.dev/), which is a spec from google that aims to encode messages with a small resource footprint usage along with other modern concepts, that helps to reduce the bandwidth consumed by the remote management protocol and have a faster parsing and decoding.

## Topology

<figure><img src="https://usp.technology/specification/architecture/usp_architecture.png" alt=""><figcaption><p>Figure 1 – USP Agent and Controller Architecture</p></figcaption></figure>

![USP training slides](https://github.com/leandrofars/oktopus/assets/83298718/b1d5a0c7-4567-464c-bc9b-1956ef5c5f3b)

![USP training slides](https://github.com/leandrofars/oktopus/assets/83298718/7b46dc1f-5eb2-4a1b-8e77-376b0836948a)

## Protocols

The [Message Transfer Protocol (MTP) ](mtps/)can be STOMP, Websockets or MQTTv5 and MQTTv3.1.1. The data is criptographed with TLS or DTLS and the messages are exchanged with [ProtoBuff](https://protobuf.dev/) encoding, having a USP Record that encapsulates a USP Message.

![image](https://github.com/leandrofars/oktopus/assets/83298718/9b789f0b-cb0c-4cec-8b8e-767ba21bffae)

## Notifications and Data Collection

You can create notifications that fire on a value change, object creation and removal, complete operation, or an event. It's also possible to create bulk data instances to send data periodically from a device to a USP Controller.

![USP training slides](https://github.com/leandrofars/oktopus/assets/83298718/184899a3-52e7-491a-8ee7-7b442fe50719)

## [OBUSPA](https://github.com/BroadbandForum/obuspa) **(Open Broadband User Services Platfrom Agent):**

Open Broadband-User Services Platform-Agent (OB-USP-Agent) is an open source project that is focused on creating a reference implementation of the User Services Platform (USP) [specification](https://usp.technology/) from an "Agent" perspective.

* Designed for embedded software (\~400kb on ARM)
* Encoded in C
* License [BSD 3-Clause](https://opensource.org/license/bsd-3-clause/)
* Made for Linux environments

## Data Analysis

The protocol has a mechanism called "Bulk Data", where it is possible to collect large volumes of data from the device, the data can be collected by HTTP, or another telemetry MTP defined in the TR standard, this data can be in JSON, CSV format or XML. This generates the opportunity to use AI on top of this data, obtaining relevant information that can be used for different purposes, from predicting events, KPIs, information for the commercial area, but also for the best configuration of a device.

## WiFi

It has over 130 Wi-Fi configuration and diagnostics metrics, many of these settings and parameters are a trade-off between signal coverage area, latency and throughput. When deploying Wi-Fi systems, there is a tendency to maintain the same configuration on all clients, causing the technology to perform below expectations. Machine Learning combined with the data analysis mentioned in the previous topic makes it possible to automate the management and optimization of Wireless networks, where a big data approach is able to find the ideal configuration for each device.

![image](https://github.com/leandrofars/oktopus/assets/83298718/3d6fe3e8-3ca2-460b-9583-da89b42753f8)

## Commands

It is possible to perform commands remotely on the product, such as: firmware update, reboot, reset, search for neighboring networks, backup, ping, network diagnostics and many others.

## IoT

\
\


<figure><img src="https://github.com/leandrofars/oktopus/assets/83298718/73e2e360-d53e-494e-9a50-60c83dae75df" alt=""><figcaption><p>Figure 34 – IoT proxied device model</p></figcaption></figure>

<figure><img src="https://github.com/leandrofars/oktopus/assets/83298718/91a87f43-3de7-42bd-a689-a4e14eecf5c0" alt=""><figcaption><p>Figure 33 – IoT individual device models</p></figcaption></figure>

<figure><img src="https://github.com/leandrofars/oktopus/assets/83298718/a2a12d9d-05a0-428b-ba3f-1ad83c876301" alt=""><figcaption><p>Figure 32 – IoT Data Model</p></figcaption></figure>

## Software Modules

Currently, telecommunications giants and startups, publishing new software daily, slow delivery cycles and manual and time-consuming quality assurance processes make it difficult for integrators and service providers to compete. USP "Software Module Management" allows a containerized approach to the development of software for embedded devices, making it possible to drastically reduce the chance of error in software updates, it also facilitates the integration of third parties in a device, still keeping the firmware part isolated from Vendor.\


<figure><img src="https://github.com/leandrofars/oktopus/assets/83298718/64664b0e-81cd-4a29-bbc5-b4186a04dfa2" alt=""><figcaption><p>Figure 28 – Possible Multi-Execution Environment Implementation</p></figcaption></figure>

