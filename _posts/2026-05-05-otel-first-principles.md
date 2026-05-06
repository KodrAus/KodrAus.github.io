---
layout: post
title: "OpenTelemetry signals from first principles"
date: 2026-05-05 09:00:00 +1000
categories: opentelemetry
---

**1:1**{: class="text-weak"} When something notable happens in your system, you emit an event for it. An event has a timestamp that tells you when it happened. It has a user-facing message that tells you what happened. It has contextual data that tells you where and why it happened.

-----

**1:2**{: class="text-weak"} Some events are more important than others so you categorise them by their severity. Higher severity events can be distinguished from, and handled differently to, lower severity ones.

<figure>
  <img class="light-only" src="https://raw.githubusercontent.com/KodrAus/KodrAus.github.io/refs/heads/master/assets/2026-05-05-otel-first-principles-logs-light.svg" alt="Three log events in a stream showing their timestamp, level, service, and message"/>
  <img class="dark-only" src="https://raw.githubusercontent.com/KodrAus/KodrAus.github.io/refs/heads/master/assets/2026-05-05-otel-first-principles-logs-dark.svg" alt="Three log events in a stream showing their timestamp, level, service, and message"/>
  <figcaption>Events can be visualised as a stream, ordered by their timestamps. The error is called out more prominently.</figcaption>
</figure>

-----

**2:1**{: class="text-weak"} Your system is organised into procedures, both logically, and mechanically. Procedures take time, and that's useful information, so you emit an event when they complete with both the time they started, and the time they ended. That lets you calculate how long they took.

<figure>
  <img class="light-only" src="https://raw.githubusercontent.com/KodrAus/KodrAus.github.io/refs/heads/master/assets/2026-05-05-otel-first-principles-spans-light.svg" alt="Three span events in a stream showing their timestamp, status, and duration"/>
  <img class="dark-only" src="https://raw.githubusercontent.com/KodrAus/KodrAus.github.io/refs/heads/master/assets/2026-05-05-otel-first-principles-spans-dark.svg" alt="Three span events in a stream showing their timestamp, status, and duration"/>
  <figcaption>Adding a second timestamp to events changes them from representing a point in time to representing a span of time.</figcaption>
</figure>

-----

**2:2**{: class="text-weak"} You want to correlate events emitted by the same procedure, so you add a [shared identifier](https://opentelemetry.io/docs/concepts/context-propagation/) to them. Procedures call eachother, so you organise their identifiers into a hierarchy. The caller becomes the parent, and callees become the children.

<figure>
  <img class="light-only" src="https://raw.githubusercontent.com/KodrAus/KodrAus.github.io/refs/heads/master/assets/2026-05-05-otel-first-principles-trace-light.svg" alt="A trace of eight spans arranged in a tree showing their span ids and relative duration"/>
  <img class="dark-only" src="https://raw.githubusercontent.com/KodrAus/KodrAus.github.io/refs/heads/master/assets/2026-05-05-otel-first-principles-trace-dark.svg" alt="A trace of eight spans arranged in a tree showing their span ids and relative duration"/>
  <figcaption>Using hierarchical identifiers lets us visualise traces as a tree of who-called-who. The timespans on events show the relative cost of each call, and how they parallelise.</figcaption>
</figure>

-----

**2:3**{: class="text-weak"} Your system makes inter-process calls that are logically part of the same procedure. You want these to be correlated too, so you include the caller's [identifier as a header](https://www.w3.org/TR/trace-context/) when making remote calls. The callee then reconstitutes it as if they were called directly in-process.

-----

**2:4**{: class="text-weak"} Tracing procedures is expensive, both in production, and in retention. Instead of recording a trace for every one, you retain a subset that sufficiently represents the whole. You [decide to trace a procedure](https://opentelemetry.io/docs/concepts/sampling/) upfront before its outer-most call, or you defer the decision to some later point when events from the trace are available. You still retain other events emitted by procedures even if their traces are not recorded.

-----

**3:1**{: class="text-weak"} Your system uses resources that you want to track over time and correlate with other behaviour. You don't know when usage changes, you can only ask for its current value, so you sample it at regular intervals and emit that value as an event. The frequency of sampling trades volume for resolution. You [aggregate samples](https://en.wikipedia.org/wiki/Aggregate_function) to preserve different properties of the underlying distribution.

<figure>
  <img class="light-only" src="https://raw.githubusercontent.com/KodrAus/KodrAus.github.io/refs/heads/master/assets/2026-05-05-otel-first-principles-chart-light.svg" alt="A line chart showing the mean, min, and max together"/>
  <img class="dark-only" src="https://raw.githubusercontent.com/KodrAus/KodrAus.github.io/refs/heads/master/assets/2026-05-05-otel-first-principles-chart-dark.svg" alt="A line chart showing the mean, min, and max together"/>
  <figcaption>Samples can be visualised as charts. This one is a line chart where the x-axis is time, and the y-axis is value, for example memory usage. These samples may have been collected at an interval of 1s, and re-aggregated with the mean, min, and max at a coarser granularity. This could happen on the producer of the samples, or when processing them later. Including the min and max gives you a sense of how the mean is skewed.</figcaption>
</figure>

-----

**3:2**{: class="text-weak"} When events are high volume, or you're particularly interested in their frequency, you count their occurrence instead of emitting them directly. You sample the count at regular intervals, and emit that value as an event.


<figure>
  <img class="light-only" src="https://raw.githubusercontent.com/KodrAus/KodrAus.github.io/refs/heads/master/assets/2026-05-05-otel-first-principles-count-light.svg" alt="A bar chart showing the count"/>
  <img class="dark-only" src="https://raw.githubusercontent.com/KodrAus/KodrAus.github.io/refs/heads/master/assets/2026-05-05-otel-first-principles-count-dark.svg" alt="A bar chart showing the count"/>
  <figcaption>Counts can be visualised as a bar chart where the x-axis is time and the y-axis is the count. Higher bars correspond to higher counts, meaning more of the event in question occurred in that interval.</figcaption>
</figure>

-----

**3:3**{: class="text-weak"} When sampling procedures that take time, you don't just want to know how many calls were made, you also want to know how long they took. You [divide the total count into buckets](https://en.wikipedia.org/wiki/Heat_map) by duration, with procedures taking about the same time sharing the same bucket. Bucket granularity trades volume for resolution.

<figure>
  <img class="light-only" src="https://raw.githubusercontent.com/KodrAus/KodrAus.github.io/refs/heads/master/assets/2026-05-05-otel-first-principles-heatmap-light.svg" alt="A bar chart showing the count"/>
  <img class="dark-only" src="https://raw.githubusercontent.com/KodrAus/KodrAus.github.io/refs/heads/master/assets/2026-05-05-otel-first-principles-heatmap-dark.svg" alt="A bar chart showing the count"/>
  <figcaption>Adding an extra dimension to a bar chart can turn it into a heatmap. In this chart, the x-axis is time, the y-axis is the span duration, and the z-axis (shading intensity) is the count of spans in that bucket. More prominently shaded regions correspond to higher counts, meaning more spans completed in about that duration, at about that time.</figcaption>
</figure>

-----

Good diagnostics compress and externalise the state of your systems with sufficient detail to understand and manage them from the outside. We've just covered the three main [observability signals](https://opentelemetry.io/docs/concepts/signals/) that make up OpenTelemetry; **1**{: class="text-weak"} [logs](https://opentelemetry.io/docs/concepts/signals/logs/), **2**{: class="text-weak"} [traces](https://opentelemetry.io/docs/concepts/signals/traces/), and **3**{: class="text-weak"} [metrics](https://opentelemetry.io/docs/concepts/signals/metrics/). I've presented them as a neat series of incremental progressions over events, but the real history isn't so serial. Each one has evolved to serve different needs at different times. Many systems today make use of all of these generic signals, plus others that may be more specialised. Over time, those specialisations that prove more broadly valuable find their way into the general toolkit, and the list grows.
