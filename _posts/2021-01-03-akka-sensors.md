---
title: "Observability of a reactive system: Introducing Akka Sensors"
excerpt: "Inline explicit instrumentation of Akka with Prometheus, with negligible overhead, open-source (MIT) and suitable for production use."
header:
#teaser: "assets/images/markup-syntax-highlighting-teaser.jpg"
tags:
- akka
toc: true
---

## Runnables
Any reactive system, [by definition](https://www.reactivemanifesto.org/), wants to be responsive, resilient and elastic, and therefore can't be anything but message-driven. 
Any action within such system is an asynchronous message that needs to be processed. 
This takes a form of a `Runnable`: data (message) + a snippet of code that needs to be executed to process the data.

## Dispatchers

Therefore, at the heart of any reactive system, there is one or more *execution contexts*.
An execution context, backed by some sort of worker pool, accepts the runnable in its inbound queue. 
When one of the context workers becomes available, the `Runnable` is executed.

In Akka runtime, a `Runnable` could be actor mailbox, processing one message, or just `ExecutionContext` that handles futures or IO monad materialisation.

It is important to keep the number of worker to a healthy minimum: keeping too many idle workers incurs overhead. 
Assigning them to CPU cores and removing one worker to assign another involves core context switch and is even more expensive.
Hence, in an optimised low-latency reactive system, limited number of workers are occupying the cores all the time, without switching, 
and just keep processing `Runnable`s as they come.

To illustrate The following 'supermarket' methaphor could be applied.

![Freepik image](/assets/images/supermarket.jpg)
<sup>(image courtesy of Freepik)</sup>

 - `Runnable`s are customers
 - worker threads are the cashiers
 - CPU cores are the cash registers

If you are efficient supermarket manager, you want your customers to spend as little time in a queue as possible. 

You could achieve that by opening many cash registers, but these are expensive and having them idle wastes money.
So instead, you just open few of them and watch the following parameters:
- how much time a customer waited in a queue before being served? (queue time)
- how much time a customer spent being served by cashier? (run time)
- how many workers are actually occupied serving customers? (active workers)

By tuning down the number of the workers to the necessary minimum, optimal latency is achieved.

There also can be situations when in an otherwise balanced shop, one customer is having issues with his payment and needs to call her husband to top her card balance.
This blocks the cachier - he can't do anything but wait - and all customers behind her, until she could pay (as you may have guessed, in a reactive system, it is a `Runnable` that behaves non-reactively, blocking thread for I/O or a mutex/lock).
We should't call the police yet (kill the thread), but a note must be made of that customer and her behaviour. 
By tracking such 'misbehaving' customers and correcting them, we could ensure that our cashiers are never blocked. 

## Actors

In a reactive system, actors are stateful entities, forming transaction boundaries and guarding state integrity.

To understand the behaviour of the system under load, the following characteristics of an actor (a class of actors or any other group) must be monitored:
- how many of a given actor (class) instances are active in memory and for how long? how often are actors passivated (receive timeout triggered)?
- how much traffic does an actor (class) gets?
- how much time an actor (class) spends on processing a message?
- how many errors and unprocessed messages are there?

In addition, for persistent actors, that are recovered from durable storage:
- how many recovery events were replayed to recover actor (class) and how much time did it take?
- how much time a persistence actor (class) spends persisting an event?

This is essential information, that will allow engineers identifying poor performance, excess resource use and root causes of incidents.

## Existing solutions for Akka

Akka itself is free open source software at its core (Apache license), but to use the great [Cinnamon telemetry library](https://developer.lightbend.com/docs/telemetry/current/home.html), commercial Lightbend subscription is required, which may not be affordable for every team.

As alternative, there is a metrics library called [Kamon](https://kamon.io/), using attached agent and bytecode instrumentation, to expose Akka metrics. Kamon is suitable for testing and moderate loads, but by nature of its instrumentation, it adds too much overhead that can't be acceptable in high-load production environments.

Teams working with Akka usually chose to implement their own custom metrics for Akka in production.

## Akka Sensors

[Akka Sensors](https://github.com/jacum/akka-sensors) is a new free open source (MIT) Scala library that instruments JVM and Akka explicitly, using native Prometheus collectors, with very low overhead, for high-load production use.

The sources are [published on Github](https://github.com/jacum/akka-sensors).

It is a greenfield implementation, not based on either Cinnamon or Kamon.

While the library itself is new as a package, the approaches and techniques applied are distilled from many years of production experience, implementing ad-hoc custom Akka/Prometheus metrics development, and from some OSS projects.

Key measurements performed on running actors, Akka dispatchers, Cassandra client. An optional 'runnable watcher', configurable per dispatcher, keeps an eye on runnables, reporting stack traces of those rogue ones, hanging too long on non-reactive activities: e.g. waiting for locks, or doing blocking I/O.

Collected metrics are indeed not as extensive as with Cinnamon, at the moment, most notably lacking the automatic instrumentation for cluster/sharding and remote traffic between cluster nodes.

It comes with set of pre-configured Grafana boards and example application ([http4s API](https://http4s.org) + Akka + [Cassandra](https://cassandra.apache.org/)).

Actor dashboard:
![Actor dashbord](https://github.com/jacum/akka-sensors/raw/master/docs/akka-actors.png)

Dispatcher dashboard:
![Dispatcher dashbord](https://github.com/jacum/akka-sensors/raw/master/docs/akka-dispatchers.png)

## References

### Akka
Akka appeared in 2009. At its core is an implementation of actor model, as known from Erlang, rewritten in Scala. Actors are stateful entities, communicating with each other asynchronously, by passing messages around. Each actor is guaranteed to process just one message a time, allowing for lock-free mutable state updates.

On top of actors keeping their state in memory, there is Akka Persistence, adding robust event sourcing, and Akka Cluster with Sharding to distribute persistent actor on available cluster nodes. Backed by scalable database such as Cassandra and scalable streaming such as Kafka, the result is a platform for nearly-infinite scalable system.

It took few years to mature into industrial quality software, and now Akka is being successfully used highly concurrent event processing systems across wide variety of industries: from gambling to banking, and from postal logistics to IoT - where each millisecond in latency matters, and data is extremely valuable. Scaling such a system to process 10x times the current load is solved by adding hardware, but, generally, without rewriting any code.

### Prometheus

Prometheus is free open source (Apache) time-series database that is widely used to keep process metrics. 

Prometheus collectors for JVM could be enhanced with any kind of metrics to collect, using very low-overhead concurrent JVM primitives.
