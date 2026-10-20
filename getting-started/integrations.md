# Integrations

Alright, you've just set up Oktopus to manage all your multi-vendor devices, hoping it to be a one platform, with one UI to solve all your problems, but then suddenly you realize that you have other systems and software for different purposes, but you wish all those solutions to converge into a cohesive ecosystem.

For this matter, Oktopus provides some alternatives:

### REST API

To integrate Oktopus into your ecosystem, you can rely on [Oktopus Controller REST API](https://api.oktopus.app.br). It's documented with Bruno, with many usage examples in different programming languages, and lots of possibilities.&#x20;

{% embed url="https://youtu.be/HXxYKSMdx_E?si=8IXs9Np5tQTGJ3HQ" %}

#### API versions

The REST API is versioned per request: every route keeps its path, and the client says which version it wants in a header.

```
X-Api-Version: 2
Accept: application/json; version=2
Content-Type: application/json; version=2
```

* A request without a version is served as **version 1**, so integrations written before versioning keep working unchanged. To be safe from a future change of the default, send `X-Api-Version: 1` explicitly.
* `X-Api-Version` takes precedence over `Accept`, which takes precedence over `Content-Type`. Different versions in two of them return HTTP `400`; a version the server doesn't serve returns HTTP `406` with the list of `supported_versions`.
* Every response carries `X-Api-Version` with the version that served it, also as the `version` parameter of its `Content-Type`.
* The Oktopus web interface always uses the latest version.

| Version     | Devices endpoints (`GET /api/device`, `GET /api/device?id=`)                                                                                                                                                                                                                                                   |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1 (default) | One row per protocol, as before devices were unified: a TR-369 device has `SN` = its Agent Endpoint ID and `sn_v2` = its serial number; a device on both protocols is two rows. The `protocol` filter accepts `tr069` and `tr369`.                                                                            |
| 2           | One row per device ([TR-069 + TR-369](enterprise-edition/scripts.md#devices-on-tr-069-and-tr-369)): `SN` is the serial number, the Agent Endpoint ID is in `usp_endpoint_id`, plus `protocol` (`tr069` \| `tr369` \| `dual-stack`) and `connections` (status and last seen per protocol). The `protocol` filter also accepts `dual-stack`. |

Everything else behaves the same on both versions:

* Device routes (`/api/device/{id}/...`) accept the serial number, `sn_v2` or the Agent Endpoint ID as `{id}`. With `any` as the protocol segment, TR-369 is used when it is online and TR-069 otherwise; each response carries `X-Oktopus-Protocol` (`tr069` \| `tr369`) and, when TR-369 didn't answer and TR-069 did, `X-Oktopus-Protocol-Fallback`. Forcing a protocol the device doesn't have returns HTTP `400` `Device has no TR-069 connection` / `Device has no TR-369 connection`; no connection available returns HTTP `400` `Device is Offline`.
* With the `protocol` filter, `status` is the status of that protocol's connection (e.g. `protocol=tr369&status=2`).
* The CSV export (`GET /api/device?...&csv=true`) has one row per device, `SN` = serial number and `Agent Endpoint ID` = Endpoint ID.

### NATS

For advanced users who may need real-time information and have experience with PUB/SUB messaging concepts, it's also possible to use NATS.

### MCP Server

Available in the Commercial Edition, it is a standardized way to connect AI assistants to your application, allowing intelligent systems to interact seamlessly with your data and functionality. Unlike direct REST API calls or direct messaging connections, [Oktopus MCP server](enterprise-edition/mcp-server.md) acts as an intelligent intermediary that exposes your application's features as discoverable "tools" that AI models can understand and use naturally within conversations.
