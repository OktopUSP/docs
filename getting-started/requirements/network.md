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

# Network

Oktopus Controller Endpoints:

<table><thead><tr><th>Endpoint</th><th>Network Protocol</th><th width="280">Application</th><th>Expose</th></tr></thead><tbody><tr><td>:9292/acs</td><td>HTTP</td><td>Auto Configuration Server (ACS)</td><td>Public</td></tr><tr><td>:8883</td><td>MQTTS</td><td>MQTT  MTP with TLS</td><td>Public</td></tr><tr><td>:1883</td><td>MQTT</td><td>MQTT  MTP without TLS</td><td>Public</td></tr><tr><td>:61613</td><td>STOMP</td><td>STOMP MTP</td><td>Public</td></tr><tr><td>:8080</td><td>Websockets</td><td>Websockets MTP</td><td>Public</td></tr><tr><td>:9443</td><td>HTTP</td><td>Portainer</td><td>Public</td></tr><tr><td>:8004</td><td>HTTP</td><td>File Server</td><td>Public</td></tr><tr><td>:80</td><td>HTTP</td><td>Nginx</td><td>Public</td></tr><tr><td>:3000</td><td>HTTP</td><td>Frontend</td><td>Public</td></tr><tr><td>:8000</td><td>HTTP</td><td>Controller REST API</td><td>Public</td></tr><tr><td>:27017</td><td></td><td>MongoDB</td><td>Internal</td></tr><tr><td>:4222</td><td>NATS</td><td>NATS Message System</td><td>Internal</td></tr></tbody></table>

Make sure that you have all the ports above available.





