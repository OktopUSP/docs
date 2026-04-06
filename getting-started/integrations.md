# Integrations

Alright, you've just set up Oktopus to manage all your multi-vendor devices, hoping it to be a one platform, with one UI to solve all your problems, but then suddenly you realize that you have other systems and software for different purposes, but you wish all those solutions to converge into a cohesive ecosystem.

For this matter, Oktopus provides some alternatives:

### REST API

To integrate Oktopus into your ecosystem, you can rely on [Oktopus Controller REST API](https://documenter.getpostman.com/view/18932104/2s93eR3vQY). It's documented in Postman, with many usage examples in different programming languages, and lots of possibilities.&#x20;

{% embed url="https://youtu.be/HXxYKSMdx_E?si=8IXs9Np5tQTGJ3HQ" %}

### NATS

For advanced users who may need real-time information and have experience with PUB/SUB messaging concepts, it's also possible to use NATS.

### MCP Server

Available in the Commercial Edition, it is a standardized way to connect AI assistants to your application, allowing intelligent systems to interact seamlessly with your data and functionality. Unlike direct REST API calls or direct messaging connections, [Oktopus MCP server](enterprise-edition/mcp-server.md) acts as an intelligent intermediary that exposes your application's features as discoverable "tools" that AI models can understand and use naturally within conversations.
