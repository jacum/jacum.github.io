---
title: "Akka Sensors: open-source observability for Akka"
excerpt: "Inline explicit instrumentation of Akka with Prometheus, with negligible overhead, suitable for production use."
header:
#teaser: "assets/images/markup-syntax-highlighting-teaser.jpg"
tags:
- akka
toc: true
---

# Observability of Akka
Observability is the capability of the system to expose its internals in a meaningful way. Akka itself is free open source software at its core (Apache license), but to use the great Cinnamon telemetry library, commercial Lightbend subscription is required, which may not be affordable for every team.

As alternative, there is a metrics library called Kamon, using attached agent and bytecode instrumentation, to expose Akka metrics. Kamon is suitable for testing and moderate loads, but by nature of its instrumentation, it adds too much overhead that can't be acceptable in high-load production environments.

Teams working with Akka usually chose to implement their own custom metrics for Akka in production.

# Akka Sensors
Akka Sensors is a free open source (MIT) Scala library that instruments JVM and Akka explicitly, using native Prometheus collectors, with very low overhead, for high-load production use.

https://github.com/jacum/akka-sensors

While the library itself is new, the approaches and techniques applied are distilled from many years of production experience, implementing ad-hoc custom Akka/Prometheus metrics development, and from some OSS projects.

Key measurements performed on running actors, Akka dispatchers, Cassandra client. An optional 'runnable watcher', configurable per dispatcher, keeps an eye on runnables, reporting stack traces of those rogue ones, hanging too long on non-reactive activities: e.g. waiting for locks, or doing blocking I/O.

Collected metrics are indeed not as extensive as with Cinnamon, at the moment, most notably lacking the automatic instrumentation for cluster/sharding and remote traffic between cluster nodes.

It comes with set of pre-configured Grafana boards and example application (http4s API + Akka + Cassandra).


# References

## Akka
Akka appeared in 2009. At its core is an implementation of actor model, as known from Erlang, rewritten in Scala. Actors are stateful entities, communicating with each other asynchronously, by passing messages around. Each actor is guaranteed to process just one message a time, allowing for lock-free mutable state updates.

On top of actors keeping their state in memory, there is Akka Persistence, adding robust event sourcing, and Akka Cluster with Sharding to distribute persistent actor on available cluster nodes. Backed by scalable database such as Cassandra and scalable streaming such as Kafka. All that makes it a platform for nearly-infinite scalable system.

It took few years to mature into industrial quality software, and now Akka is being successfully used highly concurrent event processing systems across wide variety of industries: from gambling to banking, and from postal logistics to IoT - where each millisecond in latency matters, and data is extremely valuable. Scaling such a system to process 10x times the current load is solved by adding hardware, but, generally, without rewriting any code.

## Prometheus
Prometheus is free open source (Apache) time-series database that is widely used to keep process metrics. Prometheus collectors for JVM could be enhanced with any kind of metrics to collect, using very low-overhead concurrent JVM primitives.

