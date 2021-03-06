# Summary
* [前言](README.md)
* [安全公告](security-announcements/security-announcements.md)
  * [接收安全报告](security-announcements/security-announcements.md#接收安全报告)
  * [安全性相关文档](security-announcements/security-announcements.md#安全性相关文档)
  * [修复安全缺陷](security-announcements/security-announcements.md#修复安全缺陷)
* [简介](introduction/what-is-akka.md)
  * [什么是 Akka?](introduction/what-is-akka.md)
  * [为什么使用 Akka?](introduction/why-akka.md)
  * [准备开始](introduction/getting-started.md)
  * [义不容辞的 Hello World](introduction/the-obligatory-hello-world.md)
  * [部署方案](introduction/use-case-and-deployment-scenarios.md)
  * [案例实例](introduction/examples-of-use-cases-for-akka.md)
* [概述](general/terminology-concepts.md)
  * [术语与概念](general/terminology-concepts.md)
  * [Actor Systems](general/actor-systems.md)
  * [什么是 Actor](general/what-is-an-actor.md)
  * [监管与监控](general/supervision-and-monitoring.md)
  * [Actor References, Paths and Addresses](http://doc.akka.io/docs/akka/2.4/general/addressing.html)
  * [Location Transparency](http://doc.akka.io/docs/akka/2.4/general/remoting.html)
  * [Akka and the Java Memory Model](http://doc.akka.io/docs/akka/2.4/general/jmm.html)
  * [Message Delivery Reliability](http://doc.akka.io/docs/akka/2.4/general/message-delivery-reliability.html)
  * [Configuration](http://doc.akka.io/docs/akka/2.4/general/configuration.html)
* [Actors](actors/actors.md)
  * [Actors](http://doc.akka.io/docs/akka/2.4/scala/actors.html)
  * [Akka Typed](http://doc.akka.io/docs/akka/2.4/scala/typed.html)
  * [Fault Tolerance](http://doc.akka.io/docs/akka/2.4/scala/fault-tolerance.html)
  * [Dispatchers](http://doc.akka.io/docs/akka/2.4/scala/dispatchers.html)
  * [Mailboxes](http://doc.akka.io/docs/akka/2.4/scala/mailboxes.html)
  * [Routing](http://doc.akka.io/docs/akka/2.4/scala/routing.html)
  * [FSM](http://doc.akka.io/docs/akka/2.4/scala/fsm.html)
  * [持久化](actors/persistence/0-persistence.md)
    * [事件溯源](actors/persistence/1-event-sourcing.md)
  * [Persistence - Schema Evolution](http://doc.akka.io/docs/akka/2.4/scala/persistence-schema-evolution.html)
  * [Persistence Query](http://doc.akka.io/docs/akka/2.4/scala/persistence-query.html)
  * [Persistence Query for LevelDB](http://doc.akka.io/docs/akka/2.4/scala/persistence-query-leveldb.html)
  * [Testing Actor Systems](http://doc.akka.io/docs/akka/2.4/scala/testing.html)
  * [Actor DSL](http://doc.akka.io/docs/akka/2.4/scala/actordsl.html)
  * [Typed Actors](http://doc.akka.io/docs/akka/2.4/scala/typed-actors.html)
* [Futures and Agents](futures-and-agents/futures-and-agents.md)
  * [Futures](http://doc.akka.io/docs/akka/2.4/scala/futures.html)
  * [Agents](http://doc.akka.io/docs/akka/2.4/scala/agents.html)
* [Networking](networking/networking.md)
  * [Cluster Specification](http://doc.akka.io/docs/akka/2.4/common/cluster.html)
  * [Cluster Usage](http://doc.akka.io/docs/akka/2.4/scala/cluster-usage.html)
  * [Cluster Singleton](http://doc.akka.io/docs/akka/2.4/scala/cluster-singleton.html)
  * [Distributed Publish Subscribe in Cluster](http://doc.akka.io/docs/akka/2.4/scala/distributed-pub-sub.html)
  * [Cluster Client](http://doc.akka.io/docs/akka/2.4/scala/cluster-client.html)
  * [集群分片](networking/cluster-sharding/cluster-sharding.md)
    * [实现原理](networking/cluster-sharding/how-it-works.md)
  * [Cluster Metrics Extension](http://doc.akka.io/docs/akka/2.4/scala/cluster-metrics.html)
  * [Distributed Data](http://doc.akka.io/docs/akka/2.4/scala/distributed-data.html)
  * [Remoting](http://doc.akka.io/docs/akka/2.4/scala/remoting.html)
  * [Remoting (codename Artery)](http://doc.akka.io/docs/akka/2.4/scala/remoting-artery.html)
  * [Serialization](http://doc.akka.io/docs/akka/2.4/scala/serialization.html)
  * [I/O](http://doc.akka.io/docs/akka/2.4/scala/io.html)
  * [Using TCP](http://doc.akka.io/docs/akka/2.4/scala/io-tcp.html)
  * [Using UDP](http://doc.akka.io/docs/akka/2.4/scala/io-udp.html)
  * [Camel](http://doc.akka.io/docs/akka/2.4/scala/camel.html)
* [Utilities](utilities/utilities.md)
  * [Event Bus](http://doc.akka.io/docs/akka/2.4/scala/event-bus.html)
  * [Logging](http://doc.akka.io/docs/akka/2.4/scala/logging.html)
  * [Scheduler](http://doc.akka.io/docs/akka/2.4/scala/scheduler.html)
  * [Duration](http://doc.akka.io/docs/akka/2.4/common/duration.html)
  * [Circuit Breaker](http://doc.akka.io/docs/akka/2.4/common/circuitbreaker.html)
  * [Akka Extensions](http://doc.akka.io/docs/akka/2.4/scala/extending-akka.html)
  * [Use-case and Deployment Scenarios](http://doc.akka.io/docs/akka/2.4/intro/deployment-scenarios.html)
* [Streams](streams/streams.md)
  * [Introduction](http://doc.akka.io/docs/akka/2.4/scala/stream/stream-introduction.html)
  * [Quick Start Guide](http://doc.akka.io/docs/akka/2.4/scala/stream/stream-quickstart.html)
  * [Reactive Tweets](http://doc.akka.io/docs/akka/2.4/scala/stream/stream-quickstart.html#reactive-tweets)
  * [Design Principles behind Akka Streams](http://doc.akka.io/docs/akka/2.4/general/stream/stream-design.html)
  * [Basics and working with Flows](http://doc.akka.io/docs/akka/2.4/scala/stream/stream-flows-and-basics.html)
  * [Working with Graphs](http://doc.akka.io/docs/akka/2.4/scala/stream/stream-graphs.html)
  * [Modularity, Composition and Hierarchy](http://doc.akka.io/docs/akka/2.4/scala/stream/stream-composition.html)
  * [Buffers and working with rate](http://doc.akka.io/docs/akka/2.4/scala/stream/stream-rate.html)
  * [Dynamic stream handling](http://doc.akka.io/docs/akka/2.4/scala/stream/stream-dynamic.html)
  * [Custom stream processing](http://doc.akka.io/docs/akka/2.4/scala/stream/stream-customize.html)
  * [Integration](http://doc.akka.io/docs/akka/2.4/scala/stream/stream-integrations.html)
  * [Error Handling](http://doc.akka.io/docs/akka/2.4/scala/stream/stream-error.html)
  * [Working with streaming IO](http://doc.akka.io/docs/akka/2.4/scala/stream/stream-io.html)
  * [Pipelining and Parallelism](http://doc.akka.io/docs/akka/2.4/scala/stream/stream-parallelism.html)
  * [Testing streams](http://doc.akka.io/docs/akka/2.4/scala/stream/stream-testkit.html)
  * [Overview of built-in stages and their semantics](http://doc.akka.io/docs/akka/2.4/scala/stream/stages-overview.html)
  * [Streams Cookbook](http://doc.akka.io/docs/akka/2.4/scala/stream/stream-cookbook.html)
  * [Configuration](http://doc.akka.io/docs/akka/2.4/general/stream/stream-configuration.html)
  * [Migration Guide 1.0 to 2.x](http://doc.akka.io/docs/akka/2.4/scala/stream/migration-guide-1.0-2.x-scala.html)
  * [Migration Guide 2.0.x to 2.4.x](http://doc.akka.io/docs/akka/2.4/scala/stream/migration-guide-2.0-2.4-scala.html)
* [Akka HTTP](akka-http/akka-http.md)
  * [Introduction](http://doc.akka.io/docs/akka-http/current/scala/http/introduction.html)
  * [Configuration](http://doc.akka.io/docs/akka-http/current/scala/http/configuration.html)
  * [Common Abstractions (Client- and Server-Side)](http://doc.akka.io/docs/akka-http/current/scala/http/common/index.html)
  * [Implications of the streaming nature of Request/Response Entities](http://doc.akka.io/docs/akka-http/current/scala/http/implications-of-streaming-http-entity.html)
  * [Low-Level Server-Side API](http://doc.akka.io/docs/akka-http/current/scala/http/low-level-server-side-api.html)
  * [High-level Server-Side API](http://doc.akka.io/docs/akka-http/current/scala/http/routing-dsl/index.html)
  * [Server-Side WebSocket Support](http://doc.akka.io/docs/akka-http/current/scala/http/websocket-support.html)
  * [Consuming HTTP-based Services (Client-Side)](http://doc.akka.io/docs/akka-http/current/scala/http/client-side/index.html)
  * [Server-Side HTTPS Support](http://doc.akka.io/docs/akka-http/current/scala/http/server-side-https-support.html)
  * [Handling blocking operations in Akka HTTP](http://doc.akka.io/docs/akka-http/current/scala/http/handling-blocking-operations-in-akka-http-routes.html)
  * [Migration Guides](http://doc.akka.io/docs/akka-http/current/scala/http/migration-guide/index.html)
* [HowTo: Common Patterns](howto-common-patterns/howto-common-patterns.md)
  * [Throttling Messages](http://doc.akka.io/docs/akka/2.4/scala/howto.html#throttling-messages)
  * [Balancing Workload Across Nodes](http://doc.akka.io/docs/akka/2.4/scala/howto.html#balancing-workload-across-nodes)
  * [Work Pulling Pattern to throttle and distribute work, and prevent mailbox overflow](http://doc.akka.io/docs/akka/2.4/scala/howto.html#work-pulling-pattern-to-throttle-and-distribute-work-and-prevent-mailbox-overflow)
  * [Ordered Termination](http://doc.akka.io/docs/akka/2.4/scala/howto.html#ordered-termination)
  * [Akka AMQP Proxies](http://doc.akka.io/docs/akka/2.4/scala/howto.html#akka-amqp-proxies)
  * [Shutdown Patterns in Akka 2](http://doc.akka.io/docs/akka/2.4/scala/howto.html#shutdown-patterns-in-akka-2)
  * [Distributed (in-memory) graph processing with Akka](http://doc.akka.io/docs/akka/2.4/scala/howto.html#distributed-in-memory-graph-processing-with-akka)
  * [Case Study: An Auto-Updating Cache Using Actors](http://doc.akka.io/docs/akka/2.4/scala/howto.html#case-study-an-auto-updating-cache-using-actors)
  * [Discovering message flows in actor systems with the Spider Pattern](http://doc.akka.io/docs/akka/2.4/scala/howto.html#discovering-message-flows-in-actor-systems-with-the-spider-pattern)
  * [Scheduling Periodic Messages](http://doc.akka.io/docs/akka/2.4/scala/howto.html#scheduling-periodic-messages)
* [实验性模块](experimental-modules/experimental-modules.md)
  * [Multi Node Testing](http://doc.akka.io/docs/akka/2.4/dev/multi-node-testing.html)
  * [Actors (Java with Lambda Support)](http://doc.akka.io/docs/akka/2.4/java/lambda-actors.html)
  * [FSM (Java with Lambda Support)](http://doc.akka.io/docs/akka/2.4/java/lambda-fsm.html)
  * [Persistence Query](http://doc.akka.io/docs/akka/2.4/scala/persistence-query.html)
  * [Akka Typed](http://doc.akka.io/docs/akka/2.4/scala/typed.html)
  * [外部贡献](experimental-modules/external-contributions.md)
    * [Reliable Proxy Pattern](http://doc.akka.io/docs/akka/2.4/contrib/reliable-proxy.html)
    * [Throttling Actor Messages](http://doc.akka.io/docs/akka/2.4/contrib/throttle.html)
    * [Java Logging (JUL)](http://doc.akka.io/docs/akka/2.4/contrib/jul.html)
    * [Mailbox with Explicit Acknowledgement](http://doc.akka.io/docs/akka/2.4/contrib/peek-mailbox.html)
    * [Aggregator Pattern](http://doc.akka.io/docs/akka/2.4/contrib/aggregator.html)
    * [接收流水线模式](experimental-modules/external-contributions/receive-pipeline-pattern.md)
    * [Circuit-Breaker Actor](http://doc.akka.io/docs/akka/2.4/contrib/circuitbreaker.html)
* [Information for Akka Developers](information-for-akka-developers/information-for-akka-developers.md)
  * [Building Akka](http://doc.akka.io/docs/akka/2.4/dev/building-akka.html)
  * [Multi JVM Testing](http://doc.akka.io/docs/akka/2.4/dev/multi-jvm-testing.html)
  * [I/O Layer Design](http://doc.akka.io/docs/akka/2.4/dev/io-layer.html)
  * [Developer Guidelines](http://doc.akka.io/docs/akka/2.4/dev/developer-guidelines.html)
  * [Documentation Guidelines](http://doc.akka.io/docs/akka/2.4/dev/documentation.html)
* [Project Information](http://doc.akka.io/docs/akka/2.4/project/index.html)
* [Additional Information](http://doc.akka.io/docs/akka/2.4/additional/index.html)

