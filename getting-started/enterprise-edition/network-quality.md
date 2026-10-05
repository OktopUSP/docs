---
description: >-
  The tenant-wide view of Quality of Experience: how every CPE is graded, what
  each card, chart and column of the Network Quality page means, and how to use
  it to find the CPEs that need action.
---

# Network Quality

**Network Quality** is the global view of the per-CPE [QoE Analysis](qoe-analysis.md). Every CPE is still evaluated individually, as described on the QoE Analysis page. Network Quality adds no new measurement: it puts the latest score of every CPE together so you can answer three questions quickly:

1. **How good is my network right now?**
2. **Is it getting better or worse?**
3. **Which CPEs, and which problems, need action first?**

The page has two views, switched with the buttons in the top-right corner:

* **Dashboard**: KPIs and charts for the whole network.
* **CPEs**: a table of every CPE, ranked worst first, with filters.

{% hint style="info" %}
Network Quality is in **beta**. Scores, thresholds and charts may still be fine-tuned.
{% endhint %}

## How CPEs Are Graded

Every CPE is scored individually, from 0 to 100 on each metric and on an **Overall** score, and placed in one of five grades: **Excellent** (80–100), **Good** (60–79.9), **Fair** (40–59.9), **Poor** (20–39.9) and **Bad** (0–19.9). Grey means **No data**. How each metric and the Overall score are calculated is explained in [QoE Analysis](qoe-analysis.md).

The metrics shown in the charts, filters and table columns are the same as on the QoE Analysis page: Latency, Speed, Wi-Fi Coverage, Wi-Fi Interference, Hardware, Fiber Signal, Mobile Signal, Others (stability) and WAN Errors. Only the metrics that at least one CPE in your network was scored on appear. If your fleet has no 4G/5G routers, for example, **Mobile Signal** won't appear at all.

## When Scores Are Updated

* Each CPE is re-evaluated on every collection cycle and also whenever a technician runs an **on-demand** collection. The page always shows the **latest** evaluation of each CPE (see **Last evaluated** in the table).
* A CPE counts as **evaluated** when it has at least one measurement in the evaluation window (the last 7 days by default). CPEs that were never evaluated, or that have sent nothing in that window (for example, offline for over a week), are counted as **No data**.
* A new CPE gets its first score soon after its first measurements, but trends need a few days of history. See [How Long It Takes to Appear](qoe-analysis.md#how-long-it-takes-to-appear).
* Every hour, a **snapshot** of the network summary is recorded. The "Over Time" charts are drawn from these hourly snapshots, which are kept for 90 days by default.

## Dashboard View

### KPI cards

| Card                        | What it shows                                                                                                                       |
| --------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| **Network QoE Score**       | The average **Overall** score of all evaluated CPEs, with its grade. The gauge shows the same value from 0 to 100.                   |
| **CPEs Needing Attention**  | Number of CPEs graded **Poor** or **Bad**, and their share of the evaluated CPEs. This is your action list.                         |
| **Excellent or Good**       | Share (and number) of evaluated CPEs graded **Excellent** or **Good**: the healthy part of the network.                             |
| **Total CPEs**              | Every CPE in your organization, how many of them were **evaluated**, and a line showing how the size of the network evolved.        |

The difference between **Total CPEs** and **evaluated** is the **No data** group. A large No data group means a big part of the fleet isn't reporting measurements, for example because the devices are offline or their device profile doesn't expose the needed data. See [What the CPE Needs to Have in Place](qoe-analysis.md#what-the-cpe-needs-to-have-in-place).

{% hint style="info" %}
The Network QoE Score is an **average**, so a high score can coexist with a few very bad CPEs. Always look at **CPEs Needing Attention** too.
{% endhint %}

### QoE Distribution

A donut chart of **how many CPEs are in each grade right now**. The center shows the number of evaluated CPEs, and the legend lists the count and percentage of each grade, plus the CPEs with **No data**. Percentages are calculated over the evaluated CPEs only.

Use it to understand the *shape* of your network: a healthy network is mostly dark and light green with thin Fair/Poor/Bad slices.

### QoE Distribution Over Time

A stacked area chart of the **share of CPEs in each grade**, hour by hour, over the selected period (last 24 hours up to 90 days). Things to look for:

* **Red/orange areas growing**: more CPEs are degrading. Check whether it started at a specific time, such as a maintenance window, a firmware rollout or an outage in a region.
* **A sudden step that stays**: something changed permanently (new firmware, a configuration change, a new batch of CPEs).
* **Daily waves**: problems tied to the time of day, often evening congestion or Wi-Fi interference during peak hours.

### Average QoE Score Over Time

A line chart of the **network average** over the selected period:

* The solid line is the **Overall** average.
* Each dashed line is the average of one **metric** (Latency, Speed, Wi-Fi Coverage, etc.).
* The colored background bands show the grade each value falls in.

When the Overall line drops, the dashed line that drops with it shows which metric caused it.

{% hint style="info" %}
The averages on this chart are usually higher than the share of bad CPEs suggests. Most of the fleet is healthy, so a few hundred Bad CPEs barely move an average over tens of thousands. Use this chart for **trends** and the distribution charts for **how many** CPEs are affected.
{% endhint %}

### Quality by Metric

A horizontal bar per metric showing **how the CPEs are distributed across the grades on that metric alone**. This answers *"which metric is driving bad experience?"*

For example, if **Wi-Fi Coverage** shows 73% Excellent while every other metric is close to 100%, most of the bad experience in the network comes from in-home Wi-Fi, not from the access network. That points to actions like better CPE placement or mesh/extender offers, not field maintenance on the fiber.

Each bar only counts the CPEs that report that metric. Fiber Signal, for example, is calculated over ONTs only.

### Drilling down

Most dashboard elements are clickable and open the **CPEs** view already filtered:

* **CPEs Needing Attention** → CPEs graded Poor or Bad.
* A grade in **QoE Distribution** → CPEs in that grade.
* A segment of a bar in **Quality by Metric** → CPEs in that grade **for that metric**.

## CPEs View

A table with one row per evaluated CPE, **ranked by Overall score, worst first**, so the CPEs that need action are at the top.

| Column                                     | Meaning                                                                                       |
| ------------------------------------------ | --------------------------------------------------------------------------------------------- |
| **CPE ID**, **Model**, **Vendor**          | Identification of the device                                                                  |
| **Overall**                                | Grade of the Overall score                                                                    |
| **Latency** … **WAN Errors**               | Score of each metric (0–100, colored by grade). **`-`** means the CPE has no data for that metric |
| **Last evaluated**                         | When the CPE's scores were last calculated                                                    |

Use the column button above the table to show or hide columns.

### Filters

* **CPE ID**: find a specific device.
* **Vendor** / **Model**: narrow the list to one vendor or model. This is useful to tell whether a problem is specific to a hardware model or firmware.
* **Grade** and **Metric**: work together. **Grade** picks one or more grades, and **Metric** picks which score the grade applies to. With Metric = *Overall* you get CPEs by overall grade. With Metric = *Fiber Signal* and Grade = *Bad*, you get the CPEs with a failing fiber, whatever their overall grade is.
* The clear button resets all filters.

Click a CPE ID to open the device and see its detailed [QoE Analysis](qoe-analysis.md), including its alarms and score history, before deciding on an action.

## Reading the Network View

For what an individual metric score means, see [Reading the Scores](qoe-analysis.md#reading-the-scores). At the network level, also look for these patterns:

| What you see                                                        | Likely meaning / where to look                                                                                   |
| ------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| One metric much worse than the others in **Quality by Metric**      | That metric drives most of the bad experience. Focus the action there (e.g. in-home Wi-Fi vs. field maintenance). |
| **Latency** or **Speed** dropping on many CPEs at the same time      | Likely an upstream problem (congestion, routing, a test server issue) rather than individual CPEs.              |
| Bad CPEs concentrated on one **Vendor** / **Model**                  | A hardware or firmware problem of that model. Filter by model in the CPEs view to confirm.                       |
| A sudden step in **QoE Distribution Over Time** that stays           | Something changed permanently: a firmware rollout, a configuration change, a new batch of CPEs.                  |
| Many CPEs in **No data**                                             | Devices offline for over a week, or their model's device profile doesn't provide the measurements.              |
