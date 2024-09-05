---
description: Simulate an USP agent, based on OBUSPA.
---

# Agent Simulator

Agent is a piece of software that runs on the CPE or IoT device and is responsable for connecting it to the Controller (Oktopus) through USP. To simulate the behavior of a real device we can use [OBUSPA](../../usp-tr-369/architecture.md#obuspa-open-broadband-user-services-platfrom-agent) encapsulated in a docker container for better portability.

[OBUSPA](../../usp-tr-369/architecture.md#obuspa-open-broadband-user-services-platfrom-agent) based agent simulator utility packaged by Oktopus depends on initial configuration files available at [/agent](https://github.com/OktopUSP/oktopus/tree/main/agent) folder. You can edit those files to simulate more complex environments. Run the commands below inside[ /agent](https://github.com/OktopUSP/oktopus/tree/main/agent) folder, each tab shows how to connect to a different MTP:

{% tabs %}
{% tab title="MQTT" %}
```
docker run -d -v $(pwd)/agent/oktopus-mqtt-obuspa.txt:/obuspa/oktopus-mqtt-obuspa.txt --network host --name obuspa-mqtt oktopusp/obuspa:latest obuspa -r /obuspa/oktopus-mqtt-obuspa.txt -p -v4 -i lo
```
{% endtab %}

{% tab title="Websockets" %}
```bash
docker run -d -v $(pwd)/agent/oktopus-websockets-obuspa.txt:/obuspa/oktopus-websockets-obuspa.txt --network host --name obuspa-websockets oktopusp/obuspa:latest obuspa -r /obuspa/oktopus-websockets-obuspa.txt -p -v4 -i lo
```
{% endtab %}

{% tab title="STOMP" %}
```bash
docker run -d -v $(pwd)/agent/oktopus-stomp-obuspa.txt:/obuspa/oktopus-stomp-obuspa.txt --network host --name obuspa-stomp oktopusp/obuspa:latest obuspa -r /obuspa/oktopus-stomp-obuspa.txt -p -v4 -i lo
```
{% endtab %}
{% endtabs %}

### Flags:

```
--help (-h)       Displays this help
--log (-l)        Sets the destination for debug logging. Default is 'stdout'. Can also use 'syslog' or a filename
--dbfile (-f)     Sets the path of the file to store the database in (default=/usr/local/var/obuspa/usp.db)
--verbose (-v)    Sets the debug verbosity log level: 0=Off, 1=Error(default), 2=Warning, 3=Info
--prototrace (-p) Enables trace logging of the USP protocol messages
--cli (-s)        Sets the path of the Unix domain socket file used for CLI communications
--authcert (-a)   Sets the path of the PEM formatted file containing a client certificate and private key to authenticate this device with
--truststore (-t) Sets the path of the PEM formatted file containing trust store certificates
--resetfile (-r)  Sets the path of the text file containing factory reset parameters
--interface (-i)  Sets the name of the networking interface to use for USP communication
--meminfo (-m)    Collects and prints information useful to debugging memory leaks
--register (-R)   Sets the top-level data model objects to register when acting as a USP Service
--error (-e)      Enables printing of the callstack whenever an error is detected
--command (-c)    Sends a CLI command to the running USP Agent and prints the response
                  To get a list of all CLI commands use '-c help'
--plugin (-x)     Specifies the path to a shared object vendor layer plug-in
```
