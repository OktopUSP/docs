---
description: Frequently asked questions about installation.
---

# FAQ

#### Followed the installation guide but user register page does not appear?

Check if you're connected to Oktopus through the port 80 instead of 300, then make sure tha MongoDB is running, it has some issues depending on the CPU architecture which makes it crash, so if that's the case with you, change the [docker-compose file](https://github.com/OktopUSP/oktopus/blob/main/deploy/compose/docker-compose.yaml#L41) to a different MongoDB version compatible with the server hardware.&#x20;

#### What is Oktopus default user and password?

Oktopus has no default user and password, in the first run of the software it should appear a user register page, if it didn't  appear, follow the instructions mentioned in the question above.
