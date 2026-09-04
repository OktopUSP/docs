---
description: >-
  How the Bulk Manager collects fleet telemetry: what it pulls from every CPE,
  how often, how it talks to devices, and how the volume and bandwidth numbers on
  the screen are worked out.
cover: >-
  https://images.unsplash.com/photo-1714901423336-1884cd3fb50f?crop=entropy&cs=srgb&fm=jpg&ixid=M3wxOTcwMjR8MHwxfHNlYXJjaHw1fHxkYXRhJTIwcGlwZWxpbmV8ZW58MHx8fHwxNzg4NDc3MjkwfDA&ixlib=rb-4.1.0&q=85
coverY: 0
---

# Device Telemetry

The **Device Telemetry** screen controls the **Bulk Manager**: the service that quietly walks your whole fleet and collects health data from every CPE. [QoE Analysis](qoe-analysis.md) turns that data into scores and alarms. This page explains what the collector does to your devices and your network, so you can size it well.

## The big picture

Bulk Manager runs as a group of pods. One of them is automatically the **scheduler**; the others are **workers**. If the scheduler pod dies, another takes over. Adding pods adds capacity and nothing else changes.

The work is organised into **cycles**, and there are two of them, on separate clocks:

| Cycle | What it does | Rough cost |
| --- | --- | --- |
| **General Data Collection** | Reads Wi-Fi, hardware, connected clients, interfaces, fiber, cellular, ping and site survey from every device. Stores one small document per device. | Kilobytes per device. Runs often. |
| **Speed Test Collection** | Asks the CPE to run a real download and/or upload against your servers. | Megabytes to hundreds of megabytes per device, per run. Runs rarely. |

They have separate intervals, separate parallel limits and separate on/off switches, because their network cost is thousands of times apart. Turning one off does nothing to the other.

Each organization has its own queue and its own schedule. A large fleet in one organization cannot slow down a small one in another.

## How one cycle runs

{% stepper %}
{% step %}
#### The scheduler opens a cycle

When the **Interval** has passed since the last cycle started, the scheduler pulls the full device list for the organization and creates **one job per device**, offline devices included. Jobs go onto the queue at a controlled rate.
{% endstep %}

{% step %}
#### Workers drain the queue

Any worker pod can pick up any job. Across the whole organization, at most **Devices in Parallel** devices are worked on at the same time. Each job runs the complete sequence for one device.
{% endstep %}

{% step %}
#### Offline devices are cheap

If the device is not online, the worker writes a minimal record and moves on. No device traffic is generated.
{% endstep %}

{% step %}
#### Results are written in batches

Finished documents are grouped and written to the database together. Stored telemetry is kept for 30 days by default, then expires automatically.
{% endstep %}

{% step %}
#### The cycle closes

When every job is done, the cycle closes. Its real duration and device count feed the **Cycle Duration** figure on screen. The next cycle cannot start until the current one is empty, so the Interval is a floor, not a promise.
{% endstep %}
{% endstepper %}

Bulk jobs are the lowest priority traffic to a CPE. A technician opening a device page, a Script or a Service is always served first.

## How it talks to devices

This is where the real infrastructure cost lives, and it depends on the protocol.

**USP (TR-369) devices** answer over their existing connection. Each read or test is one message.

**TR-069 devices** have to be woken with a Connection Request before every session. To keep that cost down, Bulk Manager uses a **grouped session**: all the plain parameter reads for one device (hardware, connected clients, fiber, cellular, interfaces, site survey) are sent together and answered in a **single session**, so the device is woken once, not once per element.

Diagnostics are the exception. Ping and speed tests are TR-143 diagnostics: the CPE runs one at a time and reports each result on its own later session. So for a TR-069 device with everything enabled:

* 1 session for all the grouped reads
* 1 session per ping server
* 1 session for the download test
* 1 session for the upload test

The collector does not poll the ACS queue. It sends the request and waits for the answer. If the ACS reports that the device is unreachable right now, that device is set aside and picked up again on a later pass, so one slow device never blocks the cycle. Per-pod concurrency, request rate and timeouts are all tunable at deployment time.

### Cost on the Oktopus side

This scales with your fleet, and it needs planning.

* **CPU and memory:** every device job parses and forwards JSON, and many run at once. A large fleet with high parallelism keeps many workers, connections and messages busy at the same time, so the Bulk Manager pods, the ACS and NATS all have to be sized for that peak. This is real capacity, not a rounding error, once you are past a few thousand CPEs.
* **Database:** storage grows with fleet size, how many elements you collect, and how often. The 30-day-by-default retention caps it.
* **Network:** the platform has to carry every device message and every speed test transfer. Parallelism decides how much of that happens at once.

## What General Data Collection pulls

Every element is one section of the same per-device document. A field only appears if the device's [profile](device-profile.md) supports it.

| Element | What it contains |
| --- | --- |
| **Ping** | Per host: success and failure counts, min / average / max RTT |
| **Connected Clients** | Every client: MAC, IP, hostname, signal (RSSI), activity, per-client traffic, mesh layout |
| **Hardware Resources** | Uptime, CPU %, memory %, temperature |
| **Site Survey** | Neighbor networks (SSID, BSSID, channel, signal, standard, width), computed noise per channel, current and recommended channel |
| **Cellular** | Signal, bands, data counters (TR-069 only) |
| **Interface Statistics** | Bytes, packets, errors and discards, sent and received, per interface |
| **Fiber** | Link status, optical TX / RX power |

Two of these carry most of the weight. **Connected Clients** is usually the largest section, and a busy home with many clients makes the document bigger. **Site Survey** is intrusive: the radio scans the spectrum and the customer's Wi-Fi is briefly disturbed, so it is restricted to a maintenance window you set (start, end, timezone). **Ping** costs about 160 bytes per host in the stored document, plus one diagnostic session per host on TR-069.

Once per device per cycle the worker also reads the device's capability list (one extra message) to know which of the above to attempt.

## What Speed Test Collection pulls

* **Download:** the CPE downloads the file at **Download Server URL**. The data moved is the size of that file.
* **Upload:** the CPE uploads a generated payload of **Upload File Size (MB)** to **Upload Server URL**.

Only the throughput, RTT and response time are stored, a few hundred bytes. The cost is the transfer itself, and it lands on your speed test servers and your transit, not on Oktopus.

{% hint style="warning" %}
Name the download file with its size, for example `200MB.file` or `1GB.bin`. The estimate on the screen reads the size out of the URL. A URL with no size in its name is counted as zero.
{% endhint %}

## How the estimates are calculated

Both tabs show the same five figures, and they recalculate live as you change the fields. The device count is your **whole fleet**, the same total the dashboard shows, because a cycle creates a job for every device.

### General Data Collection

| Figure | How it is worked out |
| --- | --- |
| **Per Cycle** | Fleet devices x average document size. The average is measured from the last 24 hours of stored data; a fresh fleet assumes 25 KB plus 160 bytes per ping server. Does not depend on the interval. |
| **Per Day** | Per Cycle x (24h / cycle time), where cycle time is the longer of the real last cycle and the interval. |
| **Per Month** | Per Day x 30. |
| **Cycle Duration** | The real duration of the last finished cycle. In practice this is set by how quickly CPEs answer and how many need a retry, not by raw parallelism. |
| **Bandwidth** | Average while the cycle runs (cycle bytes / cycle time). Small, because the documents are kilobytes. |

Document size is driven mostly by **Connected Clients** and **Site Survey**. Ping adds about 160 bytes per host.

### Speed Test Collection

| Figure | How it is worked out |
| --- | --- |
| **Per Cycle** | Fleet devices x (download file size + upload size), counting only the directions you enabled. |
| **Per Day** | Per Cycle x (24h / cycle time), where cycle time is the longer of `devices / parallel x median test duration` and the interval. Median test duration is measured (30s is assumed until there are samples). |
| **Per Month** | Per Day x 30. |
| **Cycle Duration** | `devices / parallel x median test duration`. |
| **Bandwidth** | `devices in parallel x median throughput per test` (100 Mbps assumed until measured). This is the **peak** traffic through your speed test servers at once, not an average. Size your network against this number. |

{% hint style="danger" %}
20 devices testing in parallel at 140 Mbps each is 2.8 Gbps flowing at the same time. Set **Devices Testing in Parallel** against the bandwidth you are willing to spend, not against how fast you want the cycle to finish.
{% endhint %}

## What each setting does to the numbers

| You change | Effect |
| --- | --- |
| **Interval** shorter | More cycles per day, so proportionally more total data and more device sessions per day. Per-cycle cost and per-device load are unchanged. |
| **Devices in Parallel** higher | Shorter cycle, but more load on the ACS and more traffic on your access network at the same time. |
| **Devices Testing in Parallel** higher | Directly raises peak bandwidth. This is the main safety lever for the speed test pipeline. |
| **Enable more Telemetry Elements** | Larger documents and more reads per session. Site Survey also adds a radio scan and needs the maintenance window. |
| **More ping servers** | About 160 bytes per device per cycle each, plus one more diagnostic session per device on TR-069. |
| **Larger speed test file, or both directions** | Proportionally more data moved per device per run. This is the largest cost in the whole system. |

## Per CPE Model overrides

The **Collection per CPE Model** tab changes behavior for one device model: which elements are collected, which tests run, which servers they use. This is handy for turning off Site Survey on a model with a fragile radio, or turning off speed tests on 4G CPEs with metered links. Telemetry keeps flowing for everything you did not change.

Scheduling and capacity stay global: the interval, the parallel limits and the maintenance window are shared by the whole organization, so one model cannot redefine them.
