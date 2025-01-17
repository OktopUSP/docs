---
noIndex: true
---

# MTPs

The MTP, or Message Transfer Protocol, is a protocol that operates one layer below the User Services Platform (USP). Its main function is to transport messages between USP Endpoints. The MTP essentially encapsulates the datagrams that make up the USP messages, along with essential information such as the USP version and the IDs of the source and destination Endpoints. In addition, it may contain metadata to ensure integrity, security, and the delivery of fragmented messages.

## Multiple MTPs and Configuration

Both Agents and Controllers can support multiple MTPs. When an Agent operates with multiple MTPs, it can be configured to connect to a specific Controller through more than one protocol. When an Agent sends a notification, it may use a specific MTP, try to send through all MTPs simultaneously, or send sequentially through each MTP.

### Supported Protocols

The USP specification defines different bindings, which allow specific protocols such as **CoAP** (deprecated), **WebSockets**, **STOMP**, **MQTT**, and **UDS** to carry USP records in their payloads.&#x20;

### Security :closed\_lock\_with\_key:

For USP implementations in environments **where messages are transported over the internet, the use of secure transport is mandatory**. MTPs that operate over TCP must implement at least the TLS 1.2 protocol.





