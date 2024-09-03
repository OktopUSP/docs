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

| Endpoint  | Network Protocol | Name                  |
| --------- | ---------------- | --------------------- |
| :9292/acs | HTTP             | ACS                   |
| :8883     | MQTTS            | MQTT  MTP with TLS    |
| :1883     | MQTT             | MQTT  MTP without TLS |
| :61613    | STOMP            | STOMP without TLS     |
|           |                  |                       |
|           |                  |                       |
|           |                  |                       |
| :80       | HTTP             | Nginx                 |
| :3000     | HTTP             | Frontend              |

Make sure that you have all the ports above available.





