---
layout: post
title: "OpenTelemetry from first principles"
date: 2026-05-05 09:00:00 +1000
categories: opentelemetry
---

**1:1**{: class="section-number"} When something notable happens in your system, you log an event for it. An event has a timestamp that tells you when it happened. It has a user-facing message that tells you what happened. It has contextual data that tells you where and why it happened.

**1:2**{: class="section-number"} Some events are more important that others so you categorize them by their severity. Higher severity events can be distinguished from, and handled differently to, lower severity ones.

![Three log events in a stream showing their timestamp, level, service, and message](https://raw.githubusercontent.com/KodrAus/KodrAus.github.io/refs/heads/master/assets/2026-05-05-otel-first-principles-logs.svg)

**2:1**{: class="section-number"} Your system is organised into procedures, both logically, and mechanically. Procedures take time, and that's useful information, so you stamp them with both the time they started, and the time they ended. That lets you calculate how long they took.

**2:2**{: class="section-number"} You want to correlate events raised by the same procedure, so you add a shared identifier to them. Each invocation of a procedure is assigned a unique identifier. Procedures call eachother, so you organise their identifiers into a hierarchy. The caller becomes the parent, and the callee becomes the child.

![A trace of eight spans arranged in a tree showing their span ids and relative duration](https://raw.githubusercontent.com/KodrAus/KodrAus.github.io/refs/heads/master/assets/2026-05-05-otel-first-principles-trace.svg)

**2:3**{: class="section-number"} Your system makes inter-process calls that are logically part of the same procedure. You want these to be correlated too, so you include the caller's identifier when making remote calls. The callee uses it as if they were in-process with the caller.

**2:4**{: class="section-number"} Tracing procedures is expensive, both in production, and in retention. Instead of recording a trace for every procedure call, you retain a subset that sufficiently represents the whole set. You can decide to trace a procedure probabilistically upfront before its outer-most call, retaining important events like errors regardless, or you can defer the decision to some later point when events from the trace are available.

**3:1**{: class="section-number"} Continuous data streams like resource utilisation don't naturally map onto events. You produce events from them by sampling their current value at regular intervals. The frequency of sampling trades volume for resolution. Sampling different aggregations retains different properties of the underlying distribution.

![A line chart showing the mean, min, and max together](https://raw.githubusercontent.com/KodrAus/KodrAus.github.io/refs/heads/master/assets/2026-05-05-otel-first-principles-chart.svg)

**3:2**{: class="section-number"} You compress high-volume events by counting their occurrence instead of logging them directly, making them both cheaper to produce, and to retain. You sample the count at regular intervals, just like continuous data streams.

**3:3**{: class="section-number"} When compressing procedures, you don't just want to know how many calls are made, you also want to know how long they took. You divide the total count into buckets by duration, with procedures taking about the same time sharing the same bucket. Bucket granularity trades volume for resolution.

![A heatmap chart showing the number of spans that completed within a given time bucket](https://raw.githubusercontent.com/KodrAus/KodrAus.github.io/refs/heads/master/assets/2026-05-05-otel-first-principles-heatmap.svg)

-----

Good diagnostics compress and externalise the state of your systems with sufficient detail to understand and manage them from the outside. We've just covered the three main observability signals that make up OpenTelemetry; logs (1:x), traces (2:x), and metrics (3:x). I've presented them as a neat series of incremental progressions, but the real history isn't so serial. Each one has evolved to serve different needs at different times. Many systems today make use of all of these generic signals, plus others that may be more specialised. Over time, those specialisations that prove more broadly valuable find their way into the general toolkit, and the list grows.
