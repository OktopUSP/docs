---
description: >-
  How Oktopus continuously scores connection quality per CPE, what data the
  device must expose, and how long it takes for results to show up.
---

# QoE Analysis

Quality of Experience (QoE) analysis is Oktopus's automated, always-on health monitoring for every CPE in your fleet. Instead of waiting for a customer to call in, Oktopus periodically pulls metrics straight from each device, compares them against its own recent history, health thresholds, and turns that into a single **Overall Health Score** plus specific alarms whenever something degrades.

## How It Works

{% stepper %}
{% step %}
#### Periodic Scrape

Oktopus polls each managed CPE at a fixed interval (30 minutes by default, configurable per deployment) and collects ping, throughput, connected-device, hardware, and Wi-Fi channel data.
{% endstep %}

{% step %}
#### Baseline First, Then Comparison

The very first scrape for a device only records a baseline — there's nothing to compare it against yet, so no score or alarms are produced. Starting from the **second** scrape, each new reading is compared against the device's own recent history to detect degradation.
{% endstep %}

{% step %}
#### Scoring

Each metric that has data produces a component health score (0-100). Components with no data for that device (for example, no throughput tests configured) are simply left out — they don't drag the score down or get counted.
{% endstep %}

{% step %}
#### Alarming

Whenever a metric crosses its configured threshold, Oktopus raises a contextual alarm (e.g. "Download throughput has degraded below baseline") in addition to lowering the relevant component score.
{% endstep %}

{% step %}
#### Overall Health Score

The **Overall Health Score** is the average of whichever component scores were actually calculated for that device on that scrape.
{% endstep %}
{% endstepper %}

## What Gets Analyzed

| Component            | What it measures                                                    |
| --------------------- | --------------------------------------------------------------------- |
| **Ping**               | Latency to one or more monitored hosts                                |
| **Throughput**         | Upload/download speed and RTT during speed tests                      |
| **Bad Signal**         | Wi-Fi clients connected with a weak signal (RSSI ≤ -70 dBm)           |
| **Hardware**           | CPU usage, memory usage, and temperature                              |
| **Channel Noise**      | Interference/noise on the Wi-Fi channel the device is actually using  |

The **Overall Health Score** is the plain average of the components above that had data for that device — there's no fixed "5 components always" rule, a device without speed-test data is scored on the remaining components only.

## Alarms

Alarms are generated alongside the score whenever a threshold is crossed, and are kept in a searchable history:

| Alarm | Trigger |
| --- | --- |
| `PingLatency` | Latency to a monitored host increased beyond the configured multiplier over its baseline |
| `UploadThroughputDegradation` / `DownloadThroughputDegradation` | Upload/download speed dropped below the bandwidth threshold |
| `UploadRttIncrease` / `DownloadRttIncrease` | RTT measured during a speed test increased beyond threshold |
| `PersistentBadSignal` | A connected client has shown a weak signal repeatedly, not just once |
| `HighCPUUsage` / `HighMemoryUsage` | CPU or memory usage stayed at or above the configured threshold |
| `OperatingChannelNoiseWorsened` | Noise increased on the **channel the device is currently transmitting on** — noise on a channel the CPE isn't using won't trigger this |

## What the CPE Needs to Have in Place

QoE analysis doesn't require anything beyond what Oktopus already needs to manage the device — but the specific scores you get depend on which data the CPE's [device profile](device-profile.md) actually exposes:

| To get this score... | ...the device profile must implement |
| --- | --- |
| Ping                  | `get_ping_result` / `parse_get_ping_result` — the CPE must support and be configured to run ping diagnostics |
| Throughput            | `get_speed_test_result` / `parse_get_speed_test_result` (and `set_speed_test` to trigger one) |
| Bad Signal            | `get_connected_devices` / `parse_get_connected_devices`, with each client reporting an RSSI value |
| Hardware              | `get_hwinfo` / `parse_get_hwinfo`, reporting CPU and memory usage |
| Channel Noise         | `get_site_survey_results` / `parse_get_site_survey`, reporting noise per channel, plus `get_radio` to know which channel is currently in use |

If a vendor's profile doesn't implement a given function, that component is simply skipped for devices of that model — it does not block the other components or produce a "failed" score. **If the CPE profile does not implement any of the functions, the analysis can't be completed.**

## How Long It Takes to Appear

* **First contact**: nothing is scored yet. The first scrape after a device comes online only records a baseline snapshot.
* **First score**: appears on the **next** scrape cycle after the baseline — with the default 30-minute interval, that's roughly 30-60 minutes after the device starts reporting data.
* **Trend-based alarms** (like `PersistentBadSignal`) need the condition to repeat across several scrapes before firing, so they naturally take longer to surface than one-off threshold breaches — a client with a weak signal has to be seen in that state repeatedly, not just once.
* **Baselines roll forward**: ping and throughput baselines are computed from a rolling window of recent data (7 days by default), so scores stay responsive to real, recent conditions rather than a single stale reading from weeks ago.
* **Retention**: raw time-series data expires automatically and alarm history is kept for 7 days.
