# PrplOS

## Introduction

PrplOS is an open-source operating system designed specifically for embedded devices and networks within the home broadband ecosystem. It is part of the prpl Foundation's initiative to support an open, distributed architecture for service delivery across the digital home. To know more access: [https://prplfoundation.org/](https://prplfoundation.org/)

{% hint style="info" %}
This tutorial was tested against [PrplOS release version 3.1.0](https://gitlab.com/prpl-foundation/prplos/prplos/-/releases/prplos-v3.1.0)
{% endhint %}

## Advantages

* **Interoperability**: PrplOS provides compatibility with a wide range of devices and platforms, fostering a multi-vendor environment.
* **Security**: Built with security as a priority, PrplOS offers features such as secure boot, firmware updates, and sandboxing to protect devices from potential threats.
* **Flexibility and Modularity**: Its modular architecture allows for customization, enabling developers to tailor the OS to specific device requirements and functionalities.
* **Efficiency**: Optimized for low-power and resource-constrained devices, PrplOS ensures efficient performance suited for various IoT applications.
* **Community-Driven**: As an open-source project, it benefits from community contributions, enhancing its features and ensuring rapid adaptation to new technologies.

## Installation

Can be installed on supported hardware platforms using standard interfaces like JTAG or UART. Prebuilt binaries are available for various development boards, allowing a straightforward flashing process with tools such as OpenOCD or manufacturer-specific utilities. This ensures quick deployment and reduces the need for extensive configuration.

For testing and debugging, simulation environments provide an alternative to physical devices. Using emulators like QEMU, developers can run the system in a virtual setup, enabling safe experimentation and troubleshooting. These options make installation adaptable to both hardware deployment and software development workflows.

### Quick Start

The easiest and fastest way to get everything running is to download a pre-built image from PrplOS repository releases and deploy it on your target platform. It's very straightforward to transform the image in a format to be used by your prefered hypervisor and launch a VM with Prpl as the operating system.

You get have some knowledge with linux networking and CLI, it's important for setting up the operating system network configuration to reach the outside world and to be able to access the guest host through SSH for convenience.\
\
Check more installation options and target platforms at: [https://gitlab.com/prpl-foundation/prplos/prplos/-/wikis/home](https://gitlab.com/prpl-foundation/prplos/prplos/-/wikis/home).

## Configuration

### TR-369 / USP

First, setup a command alias to facilitate the next steps.

```shell
alias usp="obuspa -f /etc/obuspa.db -c"
```

Setup new USP Controller Endpoint Id:

```shell
usp set Device.LocalAgent.Controller.1.EndpointID oktopusController
```

> "Essentially, _obuspa_ waits for up to 10 minutes for the presence of the _Device.DeviceInfo_ and _Device.Time.Status_ data model entries, ensuring that _Device.Time.Status_ is in a _Synchronized_ state.
>
> This timeout is configurable via Device.LocalAgent.X\_VANTIVA-COM\_PreConnectTimeout (value in seconds, 0 = disable) and can be modified either in the default _obuspa_ configuration file located at [/etc/config/obuspa\_param\_reset.txt](https://gitlab.com/prpl-foundation/prplos/feeds/feed_usp/-/blob/main/apps/obuspa/files/etc/config/obuspa_param_reset.txt?ref_type=heads#L59) or at runtime \[...]"

```sh
usp set Device.LocalAgent.X\_VANTIVA-COM\_PreConnectTimeout 60
```

```shell
service obuspa restart
```

In the below commands, exchange "\<oktopus-controller-host>" for the corresponding value of your scenario.

{% tabs %}
{% tab title="MQTTv5" %}
Define the Controller MTP as MQTT:

```shell
usp set Device.LocalAgent.Controller.1.MTP.1.Protocol MQTT
```

```shell
usp set Device.LocalAgent.MTP.1.Protocol MQTT
```

Create new MQTT client:

```shell
usp add Device.MQTT.Client
```

Setup MQTT client broker address:

```shell
usp set Device.MQTT.Client.1.BrokerAddress <oktopus-controller-host>
```

Associate Local Agent with MQTT Client:

<pre class="language-shell"><code class="lang-shell"><strong>usp set Device.LocalAgent.MTP.1.MQTT.Reference Device.MQTT.Client.1
</strong></code></pre>

Associate USP Controller with MQTT Client:

```shell
usp set Device.LocalAgent.Controller.1.MTP.1.MQTT.Reference Device.MQTT.Client.1
```

Configure MQTT topics:

```shell
usp set Device.LocalAgent.MTP.1.MQTT.ResponseTopicConfigured oktopus/usp/v1/controller
```

```shell
usp set Device.LocalAgent.Controller.1.MTP.1.MQTT.Topic oktopus/usp/v1/agent
```

Enable MQTT client connection:

```shell
usp set Device.MQTT.Client.1.Enable true
```
{% endtab %}

{% tab title="STOMP" %}
Define the Controller MTP as STOMP:

```shell
usp set Device.LocalAgent.Controller.1.MTP.1.Protocol STOMP
```

```shell
usp set Device.LocalAgent.Controller.1.MTP.1.Protocol STOMP
```

Setup the STOMP instance connection:

```shell
usp set Device.STOMP.Connection.1.Host <oktopus-controller-host>
```
{% endtab %}

{% tab title="Websockets" %}
```shell
// Tutorial coming soon ...
```
{% endtab %}
{% endtabs %}

***

### Device Logs

```shell
logread -f
```

or

```shell
logread -f | grep -i obuspa
```

### Clean all

In case you get stuck with an issue or misconfiguration, clean up your environment.

{% hint style="info" %}
**CAUTION**: This action will factory reset all the TR-181 datamodel.
{% endhint %}

```shell
rm /etc/obuspa.db &&
service obuspa restart
```
