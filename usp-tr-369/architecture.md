---
description: A brief explanation to the USP components and how it works.
---

# Architecture

## Topology

<figure><img src="https://usp.technology/specification/architecture/usp_architecture.png" alt=""><figcaption></figcaption></figure>

![image](https://github.com/leandrofars/oktopus/assets/83298718/b1d5a0c7-4567-464c-bc9b-1956ef5c5f3b)

![image](https://github.com/leandrofars/oktopus/assets/83298718/7b46dc1f-5eb2-4a1b-8e77-376b0836948a)

## Protocols

![image](https://github.com/leandrofars/oktopus/assets/83298718/9b789f0b-cb0c-4cec-8b8e-767ba21bffae)

## Notifications and Data Collection

You can create notifications that fire on a value change, object creation and removal, complete operation, or an event. It's also possible to create bulk data instances to send data periodically from a device to a USP Controller.

![image](https://github.com/leandrofars/oktopus/assets/83298718/184899a3-52e7-491a-8ee7-7b442fe50719)

## [OBUSPA](https://github.com/BroadbandForum/obuspa) **(Open Broadband User Services Platfrom Agent):**

* Designed for embedded software (\~400kb on ARM)
* Encoded in C
* License [BSD 3-Clause](https://opensource.org/license/bsd-3-clause/)
* Made for Linux environments

## Data Analysis

The protocol has a mechanism called "Bulk Data", where it is possible to collect large volumes of data from the device, the data can be collected by HTTP, or another telemetry MTP defined in the TR standard, this data can be in JSON, CSV format or XML. This generates the opportunity to use AI on top of this data, obtaining relevant information that can be used for different purposes, from predicting events, KPIs, information for the commercial area, but also for the best configuration of a device.

## WiFi

It has over 130 Wi-Fi configuration and diagnostics metrics, many of these settings and parameters are a trade-off between signal coverage area, latency and throughput. When deploying Wi-Fi systems, there is a tendency to maintain the same configuration on all clients, causing the technology to perform below expectations. Machine Learning combined with the data analysis mentioned in the previous topic makes it possible to automate the management and optimization of Wireless networks, where a big data approach is able to find the ideal configuration for each device.

![image](https://github.com/leandrofars/oktopus/assets/83298718/3d6fe3e8-3ca2-460b-9583-da89b42753f8)

*   **Commands:**

    It is possible to perform commands remotely on the product, such as: firmware update, reboot, reset, search for neighboring networks, backup, ping, network diagnostics and many others.
