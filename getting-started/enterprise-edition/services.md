---
description: A set of parameters to define the CPE configuration.
---

# Services

The services are used in conjunction with TR-x69 discovery mechanisms to achieve Zero Touch Provisioning flows. The CPE is added to the system beforehand, and the proper service is attached to it. When the field technician plugs the CPE into the network infrastructure, the CPE is provisioned with the parameters defined in the ACS specific to that customer.

## Service Definition

A service definition is a set of parameters, variables, and operations that dictate the communication structure with the CPE when applying the service to its configuration.

## Service Instances

The service instance is the attachment of a service to a CPE, defining the specific variable values for that equipment. This will be applied the first time the CPE connects to the platform and any time it is factory reset.

## In Practice

#### Configure the CPE to connect to the ACS

The CPE must have a way to connect to the Controller system. You can use one of these options:

* Firmware with ACS pre-configured
* DHCP with option 43
* PPPoE reply with ACS server URL

{% stepper %}
{% step %}
### Add a CPE

Manually insert a new device with its unique identifier (e.g, serial number, agent endpoint id).
{% endstep %}

{% step %}
### Create a service definition

Write the Lua scripts and necessary variables to apply that service to the CPE.
{% endstep %}

{% step %}
### Attach the service to the CPE

Associate the CPE with a service, with it's specifc variables values for that device.
{% endstep %}

{% step %}
### Plug the CPE into the network

As the CPE already has the ACS configured and a connection to reach it, right at the time it establishes communication with the controller system, all the services will be provisioned, and the device will come online with the customer's specific configuration.
{% endstep %}
{% endstepper %}
