---
description: Suited for production usage, critical workloads and commercial use.
---

# Enterprise

Find flexible pricing and resources tailored to fit your unique needs and usecase. Email us: [sales@oktopus.app.br](mailto:sales@oktopus.app.br) to get a quote or [schedule a meeting](https://oktopus.app.br/schedule-meeting) so we can better understand your needs and find the best solution.

Check our standard pricing plans:  [https://oktopus.app.br/#offer](https://oktopus.app.br/#offer).

## Homologated Devices

| Model            | Vendor    | Protocol |
| ---------------- | --------- | -------- |
| EG8145X6         | Huawei    | TR-069   |
| EG8145X6-10      | Huawei    | TR-069   |
| WS5203-20        | Huawei    | TR-069   |
| WS7001-40        | Huawei    | TR-069   |
| DG6450A/CH       | Arris     | USP      |
| WiFiber AX 3000  | Intelbras | USP      |
| WiFiber AX 3000V | Intelbras | USP      |
| WiFiber AX 1800  | Intelbras | USP      |
| WiFIber 121AC    | Intelbras | TR-069   |
| WiFiber 120AC    | Intelbras | TR-069   |
| WiFiber 1200R    | Intelbras | TR-069   |
| Raspberry Pi     | RDK-B     | USP      |

{% hint style="info" %}
In case your devices are not in that list, we take one week to homologate it.&#x20;
{% endhint %}

***

## Resources

The Enterprise Edition has everything that the Community Edition already comes with, plus:

### Multi Tenancy

Oktopus can have it's core in the cloud or in a central server, with multiple instances of ACS, MQTT, Websockets and STOMP spreaded across a region, so you ensure high availability in different nodes with very small latency, since the MTPs are deployed near the device and the control plane layer is placed in a central place for multiple users access.

<figure><img src="../../.gitbook/assets/image (15).png" alt=""><figcaption><p>Multiple MTPs instances RTT</p></figcaption></figure>

Redundant connections are achieved with Oktopus USP multi-Controller/multi-MTP support and advanced configuration for ACSs, so we ensure a stable operation with enterprise-level SLA and fail-over in case of servers interruption at the edge.

### Overview Dashbaord

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

### RBAC

Each user has a role with a collection of permissions, ensuring fine grained access to the software based on each user role capabilities.

<figure><img src="../../.gitbook/assets/image (29).png" alt=""><figcaption><p>Role with Grained Permissions</p></figcaption></figure>

### File Server

Store firmware images and apps to be downloaded by CPEs and used at firmware update and lifecycle container management (LCM).

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption><p>Firmware Images and App Store</p></figcaption></figure>

## Device Management

Enterprise-grade clients have their devices homologated into the Oktopus platform, which makes possible to have a rich experience with each device having it's particular data model extracted and studied to have custom capabilites, extracting the maximum of capabilities and possible configurations.

{% hint style="info" %}
The capabilites showed below are dependent on the device capabilites created by the vendor and exposed through CWMP and USP standards.
{% endhint %}

### Wi-Fi

Manage the SSID, Password, Channel, Transmit Rate, Bandwidth and Standards of the WLAN interface.

<figure><img src="../../.gitbook/assets/image (23).png" alt=""><figcaption><p>Simple WiFi Management</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/Screenshot from 2024-08-10 14-47-40 (1).png" alt=""><figcaption><p>Advanced Wi-Fi Management</p></figcaption></figure>

### Site Survey

Scan the Wi-Fi spectrum to find the most polluted and clean channels, to configure your network in the  channel with less interference.

<figure><img src="../../.gitbook/assets/image (22).png" alt=""><figcaption><p>Site Survey</p></figcaption></figure>

### Connected Devices

Get a list of all devices connected to the CPE, including their IP source, address, MAC, hostname and RSSI. This way you can check if a connection problem is because of a too old device Wi-Fi negotiation standard, bad Wi-Fi signal or other subject.  &#x20;

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption><p>Connected Devices Signal View</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption><p>Connected Devices Topology View</p></figcaption></figure>

<figure><img src="https://lh7-rt.googleusercontent.com/slidesz/AGV_vUdzICps2-fo90VRB7iHqxbvFPurng50N3ROsKXnSNpfCjyC308qXUxleT_vi9-nBIp4Ye9D6U97CJNVTK4nGE9DueROgAfx_T3hwWDZmf4Xytz6d3QwlZ8WNHs2XcW2uN4mbaHxVptmgplEYzNMniGxGzy-8CZDHM7iW7-GdXs9Rg=s2048?key=jha3naML321i7bIkz896lw" alt=""><figcaption><p>Connected Devices List View</p></figcaption></figure>

### Historic

Time series information  about connection speed, consumed data rate, latency, behavior of connected devices and CPE interfaces.

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption><p>Latency Data</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption><p>Connected Devices RSSI</p></figcaption></figure>

### Ports

Physical ports statistics, IPv4, IPv6 and LAN/WAN status.

<figure><img src="../../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption><p>Ports Overview</p></figcaption></figure>

### Actions

Tired of asking the customer to reboot the device or execute a factory reset and then suddenly things go wrong with a broken fiber cable, lost connection and/or physical problems? Execute multiple commands in your device and have no need of customers action.

<figure><img src="https://lh7-rt.googleusercontent.com/slidesz/AGV_vUe-nBj28KBmSncRu4WY18z411qdYas2MR-mbDiBQIMZNoqr3a62w07vk6MayPRCG6rjv7AqvDAKBII9serF3E4xXhwRZmdw1VfkA4dzwQzxEfEhT_TWJy1PBHg7SkmITTBXaoeQsT8P6loD47Q6_rmCsc4W92HFg2f1KPk0zuZr=s2048?key=jha3naML321i7bIkz896lw" alt=""><figcaption><p>Available Actions in the Device</p></figcaption></figure>

### Diagnostic Tools

Troubleshoot connectivity problems using day to-day tools as ping and traceroute. Easily identifying network configuration issues.

<figure><img src="../../.gitbook/assets/image (24).png" alt=""><figcaption><p>Diagnostic Tools</p></figcaption></figure>

### Mass Actions

Operations to be applied in batch.

<figure><img src="../../.gitbook/assets/Screenshot from 2024-09-04 17-25-44.png" alt=""><figcaption><p>Mass Firmware Update Overview</p></figcaption></figure>

### Mass Firmware Update

Execute a firmware update in multiple devices at the same time. Filter the devices you wish to update the firmware and solve the problem about having multiple deprecated firmware versions in your network containing security issues, bugs and no patches.

<figure><img src="../../.gitbook/assets/image (25).png" alt=""><figcaption><p>Mass Firmware Update Resume</p></figcaption></figure>

### Mass Custom Message

Generate custom CWMP and USP messages to be applied in multiple devices at the same time. Create complex requests tailored to multiple use cases.

<figure><img src="../../.gitbook/assets/image (3) (1).png" alt=""><figcaption><p>Mass Custom Message</p></figcaption></figure>

### Geolocation

Automatic discovery of devices geolocation.&#x20;

Keep charge of the regions where you can expand your business, the places where you have more customers and troubleshoot issues related to an area, maybe caused by physical infrastructure damage, or other factors.

<figure><img src="../../.gitbook/assets/image (2) (1).png" alt=""><figcaption><p>Device Geolocation View</p></figcaption></figure>

### Custom UI Theme

Fully customizable user interface, with your company's logo and theme. Increase your customers' confidence in your solution and the credibility of your brand, with the same functionalities as the standard web interface version.

<figure><img src="../../.gitbook/assets/image (2) (1) (1).png" alt=""><figcaption><p>Custom Theme and Logo</p></figcaption></figure>



