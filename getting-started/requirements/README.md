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

# Requirements

For small deployments with 10 \~ 50 thousand devices, the standalone mode might be a good deal. If you have want to manage more than 50 thousand devices or have a business critical operation, than you should study to with the high availability mode.

On real world environments, just to have Oktopus Controller to manage your devices is not enough, you should care about many other components, as load balancing, bakups, logs, hardware metrics, DoS protection, redundancy, scalabilty, and more. All those components are not addressed into Oktopus Controller and should be taken in count to have a production-ready environment. To have all of this set, we advice you to rely on experienced professionals, with a good understanding of USP and/or CWMP standards, just as we provide in the [Enterprise Edition.](../features/#enterprise-edition)
