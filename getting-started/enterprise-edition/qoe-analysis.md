---
description: >-
  How Oktopus scores the Quality of Experience of every CPE: the grades, the
  metrics, how each score is calculated, the alarms, and what the CPE must
  report.
---

# QoE Analysis

Quality of Experience (QoE) analysis is Oktopus's automated, always-on health monitoring for every CPE in your fleet. Instead of waiting for a customer to call in, Oktopus collects metrics straight from each device, compares them with the device's own recent history and with absolute health thresholds, and turns that into a score per metric, an **Overall** score, and specific alarms whenever something degrades.

This page explains how each CPE is evaluated individually. To see the scores of all CPEs together, ranked and trended over time, see [Network Quality](network-quality.md).

## How It Works

{% stepper %}
{% step %}
#### Collection

Oktopus collects measurements from each CPE on a regular cycle: ping, speed tests, connected Wi-Fi clients, hardware usage, site survey, fiber, cellular and interface counters. What is collected, and how often, is controlled in [Device Telemetry](bulk-device-manager.md).
{% endstep %}

{% step %}
#### Evaluation

On every evaluation, Oktopus looks at the last **7 days** of measurements of the CPE. Evaluations run on a schedule (every 30 minutes by default) and also whenever a technician triggers an **on-demand** collection.
{% endstep %}

{% step %}
#### Scoring

Each metric with data gets a score from 0 to 100, by comparing the **last day** with the **rest of the week** (the trend) and by checking **absolute thresholds** (the level). The lower of the two is kept.
{% endstep %}

{% step %}
#### Overall score and alarms

The metric scores are combined into the **Overall** score. Alarms are raised for the degradations found, so you know exactly what changed.
{% endstep %}
{% endstepper %}

## The Grading Scale

Every score goes from **0 to 100** and falls into one of five grades. This five-point scale follows the ITU-T P.800 (MOS) convention, which is widely used to rate how people experience a voice or data service.

| Grade         | Score range | What it means for the subscriber                              |
| ------------- | ----------- | ------------------------------------------------------------- |
| **Excellent** | 80 – 100    | No noticeable problem                                         |
| **Good**      | 60 – 79.9   | Works well, with small issues or a value close to a limit     |
| **Fair**      | 40 – 59.9   | Noticeable degradation; the subscriber may start to complain  |
| **Poor**      | 20 – 39.9   | Clearly degraded service                                      |
| **Bad**       | 0 – 19.9    | Service severely affected or not working                      |

Colors are the same everywhere in the UI: dark green (Excellent), light green (Good), yellow (Fair), orange (Poor) and red (Bad). Grey means **No data**.

## The Metrics

A CPE only gets a score for the metrics it actually reports. For example, Fiber Signal only exists on ONTs, and Mobile Signal only on 4G/5G routers. A metric that is missing shows as **`-`**. It is never treated as 100 (a CPE that doesn't report fiber power doesn't have a "perfect" fiber) and never treated as 0 either. It just doesn't count.

| Metric (as shown in the UI) | What it measures                                                                |
| --------------------------- | ------------------------------------------------------------------------------- |
| **Latency**                 | Latency and packet loss to the test servers configured for your organization    |
| **Speed**                   | Download and upload speed, and the latency measured during the speed tests      |
| **Wi-Fi Coverage**          | Signal strength (RSSI) of the Wi-Fi clients connected to the CPE and its mesh   |
| **Wi-Fi Interference**      | Noise on the Wi-Fi channel the CPE is using                                     |
| **Hardware**                | CPU and memory usage of the CPE                                                 |
| **Fiber Signal**            | Optical power received by the ONT                                               |
| **Mobile Signal**           | 4G/5G signal of the serving cell (RSRP, SINR, RSRQ)                             |
| **Others**                  | Stability: how many times the CPE rebooted in the last 7 days                   |
| **WAN Errors**              | Share of WAN packets lost to errors and discards                                |

## How a Metric Is Scored

Every metric is looked at in **two ways**, and the **lower** of the two results is kept.

### 1. Trend: is it worse than usual for this CPE?

The **last day** of measurements is compared with the **rest of the week** for the same CPE. Speed tests and WAN counters run less often, so they use the last **two** days, so that a single bad test doesn't decide alone. Comparing a CPE with its own recent past is what lets Oktopus catch a degradation even when the absolute values still look acceptable.

* If the recent value is within a **20% margin** of the usual one, the trend score is **100**.
* Beyond that, the score drops **in proportion** to how much worse it got:
  * Latency twice as high as usual → **50**. Three times as high → **33**.
  * Speed at half of the usual → **50**.
* Very small latency changes are ignored even when the percentage looks large. Going from 3 ms to 4 ms is +33%, but no subscriber notices it, so a rise has to be at least **5 ms** to count.
* When the subscriber's **contracted plan** (hired download/upload bandwidth) is registered for the CPE, Speed is compared with the plan instead of with the previous week.
* Signal and noise levels (in dB) are compared by **difference in dB**, not by percentage:

| Measurement                   | Counts as degraded when it changes by...  |
| ----------------------------- | ------------------------------------------ |
| Fiber receive power           | 3 dB weaker (half of the light lost)       |
| Mobile RSRP                   | 6 dB weaker                                |
| Mobile SINR                   | 5 dB weaker                                |
| Mobile RSRQ                   | 3 dB weaker                                |
| Wi-Fi channel noise           | 6 dB louder                                |

For these dB measurements, and for a rise in WAN errors, a degradation takes the score down by a fixed step, into the **Good** grade (60), rather than proportionally. For Mobile Signal, RSRP, SINR and RSRQ are checked separately, and the score is the share of them that didn't degrade.

### 2. Level: is it bad no matter what?

A CPE that has *always* been bad would look fine on the trend alone, because nothing changed. To prevent that, a set of **absolute thresholds** caps the score. Once a threshold is crossed, the score cannot be higher than the middle of the corresponding grade:

| Cap          | Maximum score |
| ------------ | ------------- |
| Attention    | 75 (Good)     |
| Fair         | 55 (Fair)     |
| Poor         | 35 (Poor)     |
| Bad          | 15 (Bad)      |

| Metric           | Attention (max. Good) | Max. Fair                                       | Max. Poor     | Max. Bad      |
| ---------------- | --------------------- | ----------------------------------------------- | ------------- | ------------- |
| Latency – loss   | ≥ 2%                  | ≥ 5%                                            | ≥ 15%         | ≥ 50%         |
| Latency – delay  | ≥ 100 ms              | ≥ 200 ms                                        |               |               |
| Speed            | < 80% of the plan     | < 50% of the plan                               |               |               |
| Fiber Signal     | ≤ -25 dBm             | ≥ -7 dBm (receiver saturated by too much light) |               | ≤ -27 dBm     |
| Mobile – RSRP    | ≤ -100 dBm            | ≤ -110 dBm                                      | ≤ -120 dBm    |               |
| Mobile – SINR    | < 10 dB               | < 5 dB                                          | < 0 dB        |               |
| Mobile – RSRQ    |                       | ≤ -15 dB                                        |               |               |
| WAN Errors       | ≥ 0.1%                |                                                 | ≥ 1%          |               |

Notes:

* Latency uses the **best-responding** test server, so one far-away server doesn't penalize the CPE. A test server that never answered during the whole week, while others did, is treated as a problem of that server (wrong name, server blocking ping) and ignored. If **no** test server answered during the last day, Latency is **0**.
* The Speed caps only apply when the contracted plan is registered for the CPE.
* These thresholds were calibrated on real production fleets. For example, 99% of lines lose less than 2% of pings, so they only catch real problems.

### Metrics with their own rules

A few metrics count events rather than compare values:

* **Wi-Fi Coverage**: each client with a weak signal takes points off, and weaker clients take more off. The result is averaged over the last day, so phones coming and going don't make it jump.

  | Client signal (RSSI)   | Weight |
  | ---------------------- | ------ |
  | Better than -70 dBm    | 0      |
  | -70 to -75 dBm         | 0.5    |
  | -75 to -80 dBm         | 1      |
  | Below -80 dBm          | 1.5    |

  Each full weight costs **15 points**: no weak client is 100, two weak clients at -78 dBm is 70, and the score never goes below 10. A Wi-Fi Coverage of **10** means many clients are connected with a poor signal. This usually points to a coverage problem in the home (CPE badly placed, missing mesh/extender) rather than a network fault.
* **Wi-Fi Interference**: the noise of the channel the CPE is **currently using** is compared with the rest of the week, per band (2.4 GHz, 5 GHz...). One band 6 dB louder is **60**; two or more is **20**. Noise on channels the CPE doesn't use doesn't lower the score.
* **Hardware**: uses the **median** CPU and memory usage of the last day, so short spikes don't matter. CPU or memory at or above the high-usage limit (90% by default) is **60**; both is **20**. A reading stuck at 100% for the whole week is treated as a reporting problem of the CPE and ignored.
* **Others (stability)**: counts the reboots of the last 7 days. A reboot is detected when the uptime reported by the CPE goes back to a smaller value, whatever the cause: power outage, crash, firmware upgrade or manual reboot.

  | Reboots in 7 days | Score |
  | ----------------- | ----- |
  | 0                 | 100   |
  | 1                 | 85    |
  | 2                 | 70    |
  | 3                 | 50    |
  | 4 – 5             | 30    |
  | 6 or more         | 10    |

## The Overall Score

The **Overall** score combines the metrics of a CPE in two steps:

1. A **weighted average**. The metrics the subscriber feels directly weigh more than device internals:

   | Metric                                                      | Weight |
   | ----------------------------------------------------------- | ------ |
   | Latency, Speed, Wi-Fi Coverage, Fiber Signal, Mobile Signal | 1.0    |
   | Others (stability)                                          | 0.8    |
   | Hardware, Wi-Fi Interference, WAN Errors                    | 0.6    |

2. The result is **pulled toward the worst metric**: 70% weighted average + 30% worst metric.

The second step makes sure one serious problem can't hide behind several healthy metrics.

{% hint style="success" %}
**Example**: a CPE has Latency 100, Speed 100, Wi-Fi Coverage 100, and Fiber Signal 20 (failing fiber).

* Weighted average = (100 + 100 + 100 + 20) / 4 = 80. That alone would read as Excellent.
* Overall = 0.7 × 80 + 0.3 × 20 = **62 → Good**, which correctly flags the CPE instead of hiding the fiber problem.
{% endhint %}

If a CPE has only **one** metric with data, Overall is that metric's score. For example, a CPE whose only score is Wi-Fi Coverage = 10 has an Overall of **10 (Bad)**.

## Reading the Scores

| What you see                                    | Likely meaning / where to look                                                                                               |
| ----------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| **Wi-Fi Coverage** low                          | Many clients with a weak signal. In-home coverage problem: CPE placement, walls, missing mesh/extender.                       |
| **Wi-Fi Interference** low                      | The channel in use got noisier. Check neighboring networks and consider changing the channel.                                |
| **Latency** low                                 | Latency or packet loss got worse. If many CPEs drop together, look upstream (congestion, routing, a test server issue).      |
| **Speed** capped at Good or Fair                | Speed tests reach less than 80% (Good) or 50% (Fair) of the registered plan.                                                 |
| **Fiber Signal** Good (75)                      | Receive power close to the receiver limit (≤ -25 dBm). It works, but there's little margin left. Check the drop, splices and connectors. |
| **Fiber Signal** around 60                      | Receive power dropped 3 dB or more compared with the rest of the week: something changed on the fiber path recently.         |
| **Fiber Signal** Fair (55)                      | Receiver saturated (≥ -7 dBm, too much light). Usually a missing or wrong attenuator, or a very short link.                  |
| **Fiber Signal** Bad                            | Receive power ≤ -27 dBm, at the receiver's sensitivity limit. Field inspection needed.                                       |
| **Mobile Signal** low                           | Weak or degraded 4G/5G signal. Check the antenna position, or whether the CPE moved to a worse cell.                         |
| **Others** low                                  | The CPE rebooted several times this week: power issues, crashes or repeated manual reboots.                                  |
| **Hardware** low                                | CPU or memory near saturation most of the day. Check the firmware, the number of connected clients and the load.            |
| **WAN Errors** low                              | Packet errors/discards on the WAN interface. Often a physical-layer issue (cabling, optics, line quality).                   |

## Alarms

Alarms are raised alongside the scores, so you know *what* changed and not only that the score went down. They are kept in a searchable history.

| Alarm | Trigger |
| --- | --- |
| `PingLatency` | Latency to a test server got worse than usual (beyond the 20% margin and by at least 5 ms) |
| `DownloadThroughputDegradation` / `UploadThroughputDegradation` | Download/upload speed dropped more than 20% below the usual speed, or below the plan when one is registered |
| `DownloadRttIncrease` / `UploadRttIncrease` | The latency measured during the speed tests got worse than usual |
| `PersistentBadSignal` | A Wi-Fi client was seen with a weak signal repeatedly over the week, not just once |
| `HighCPUUsage` / `HighMemoryUsage` | The median CPU or memory usage of the last day reached the high-usage limit (90% by default) |
| `OperatingChannelNoiseWorsened` | Noise rose by 6 dB or more on the **channel the CPE is using**. Noise on a channel the CPE isn't using doesn't trigger it |

Fiber Signal, Mobile Signal, Others and WAN Errors don't raise alarms of their own yet. Their problems show up in the scores.

## What the CPE Needs to Have in Place

QoE analysis doesn't require anything beyond what Oktopus already needs to manage the device. However, the metrics you get depend on which data the CPE's [device profile](device-profile.md) exposes and which elements are enabled in [Device Telemetry](bulk-device-manager.md):

| To get this metric...  | ...the device profile must implement |
| ---------------------- | ------------------------------------ |
| Latency                | `get_ping_result` / `parse_get_ping_result`. The CPE must support ping diagnostics |
| Speed                  | `set_speed_test`, `get_speed_test_result` / `parse_get_speed_test_result` |
| Wi-Fi Coverage         | `get_connected_devices` / `parse_get_connected_devices`, with each client reporting an RSSI value |
| Wi-Fi Interference     | `get_site_survey_results` / `parse_get_site_survey`, reporting noise per channel, plus `get_radio` to know which channel is in use |
| Hardware               | `get_hwinfo` / `parse_get_hwinfo`, reporting CPU and memory usage |
| Fiber Signal           | `get_pon` / `parse_get_pon`, reporting the received optical power |
| Mobile Signal          | `get_cellular` / `parse_get_cellular`, reporting RSRP, SINR and/or RSRQ |
| Others (stability)     | `get_hwinfo` / `parse_get_hwinfo`, reporting the uptime |
| WAN Errors             | `get_statistics` / `parse_get_statistics`, reporting packets, errors and discards of the WAN interface |

If a profile doesn't implement a given function, that metric is simply skipped for devices of that model. It doesn't block the other metrics or produce a "failed" score. **If the profile implements none of them, the CPE can't be evaluated.**

## How Long It Takes to Appear

* **First score**: the CPE is scored on the first evaluation after its first measurements are collected. At that point there's no history to compare with, so only the absolute thresholds (the level) can lower the score.
* **Trends**: the trend needs measurements older than one day to compare with (two days for speed tests and WAN counters). Trend scores and trend alarms become meaningful once the CPE has a few days of history, and fully reliable after a week.
* **Persistent alarms**: `PersistentBadSignal` needs the weak signal to be seen many times over the week, so it takes longer to show up than a one-off threshold breach.
* **Rolling window**: every evaluation looks at the last 7 days only, so scores follow the CPE's real, recent conditions. A problem that is fixed stops lowering the score once the last day is healthy again. A reboot keeps counting for 7 days.
* **No recent data**: a CPE that hasn't sent any measurement in the last 7 days is shown as **No data** in [Network Quality](network-quality.md).
* **Retention**: alarm history is kept for 7 days.
