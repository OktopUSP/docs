---
description: Suited for DIY people, developers and enthusiasts
---

# Community

## Dashboard

Check the overall performance of the system, and get insights about the devices as their type, total quantity and status.

<figure><img src="../../.gitbook/assets/image (3) (1).png" alt=""><figcaption><p>Overview Section</p></figcaption></figure>

## Devices

Table list with all the devices connected to Oktopus, independent of the protocol used for management, all of them will appear in this list, with options to filter for specific devices characteristics, and remove or add table fields to view.

<figure><img src="../../.gitbook/assets/image (8) (1).png" alt=""><figcaption><p>Devices Section</p></figcaption></figure>

### Parameters Discovery

Acces the device you wish and discover the data model dinamically.

{% hint style="info" %}
This feature is only available for USP devices.
{% endhint %}

<figure><img src="../../.gitbook/assets/image (21).png" alt=""><figcaption><p>Data Model Discovery</p></figcaption></figure>

### Custom Messages

Send custom USP or CWMP native packets, It is possible to create and save messages templates to be used later, according to each protocol specification.

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption><p>Message Template Saved</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption><p>Create New Template</p></figcaption></figure>

<details>

<summary></summary>



</details>



## Credentials

This section defines credentials to be used by the devices to connect to Oktopus Controller, those credentials are going to be attached to the ACS, MQTT, STOMP and Websockets instances that will now check if the device has the correct keys.

For this to work you need to enable this feature through environment variables in each server/broker protocol you want to have authentication enabled.&#x20;

{% hint style="info" %}
The only protocol that still don't have support to this authentication process is STOMP, which follows another authentication mechanism in the moment.
{% endhint %}

<figure><img src="../../.gitbook/assets/image (12).png" alt=""><figcaption><p>Credentials Section</p></figcaption></figure>

## Users

There are two levels of users availables in the Community Edition:

* _Admin_: Have access to the hole software platform, including the rights to remove/add users and devices credentials.
* _User_: Can access devices and configure the same, view users, dashboard information and credentials, but is not allowed to remove/add other users and devices credentials.

<figure><img src="../../.gitbook/assets/image (14).png" alt=""><figcaption><p>Users Management Page</p></figcaption></figure>
