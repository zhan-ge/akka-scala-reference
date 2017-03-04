# 什么是 Akka？

**<<可恢复、弹性式、分布式、实时事务处理>>**

我们认为想要正确的编写分布式、并发、高容错、可扩展的应用是很难的。而这在大多数时候是因为我们使用了错误的工具、错误的抽象层次。Akka 的到来就是为了改变这一切。使用 Actor 模型时，我们提升了抽象层级，提供了一个更好的平台来编写可扩展、可恢复、有求必应(responsive)的应用——可以参考 [响应式宣言(Reactive Manifesto)](http://reactivemanifesto.org/) 查看具体细节。我们采用一种“让其崩溃(let it crash)”模型来实现高容错性，该模型已成功应用于电信产业以构建能够自愈且永不宕机的应用。Actor 同时提供了“透明分布(transparent distribution)”的抽象和用于构建真正可扩展、高容错应用的基础设施。

Akka 是一个开源项目，基于 Apache 2 License。

可以在这里找到指定版本的下载链接：[http://akka.io/downloads](http://akka.io/downloads)。

请注意，所有的示例代码都是编译通过的，如果想要查看源码，请查看 Akka 文档在 Github 上子项目，包含 [Java](http://github.com/akka/akka/tree/v2.4.17/akka-docs/rst/java/code/docs) 和 [Scala](http://github.com/akka/akka/tree/v2.4.17/akka-docs/rst/scala/code/docs) 两个部分。

## Akka 实现了一种独特混合

### Actors

Actor 为你提供了：

- 对分布式、并发、并行的简单、高级抽象。
- 异步、无阻塞且高性能的消息驱动编程模型。
- 非常轻量级的事件驱动(event-driven)进程(这里指 Actor，单 GB 堆内存支持数百万 Actor)。

可以参考 [*Scala*](http://doc.akka.io/docs/akka/2.4/scala/actors.html#actors-scala) 或 [*Java*](http://doc.akka.io/docs/akka/2.4/java/untyped-actors.html#untyped-actors-java) 查看具体细节。// TODO，替换翻译链接

### 容错性

- 支持 “let-it-crash” 语义的监管层级(Supervisor hierarchies)。
- Actor 系统可以跨越多个 JVM 来提供真正的高容错系统。
- 优秀的适用于编写自愈、从不宕机的高容错系统。

查看 [*Fault Tolerance (Scala)*](http://doc.akka.io/docs/akka/2.4/scala/fault-tolerance.html#fault-tolerance-scala) 或 [*Fault Tolerance (Java)*](http://doc.akka.io/docs/akka/2.4/java/fault-tolerance.html#fault-tolerance-java) 了解更多细节。// TODO，替换翻译链接

### 地理位置透明

Akka 中的一切都被设计为适用于分布式环境：Actor 的所有交互均由异步的消息传递实现。

可以在 [*Java*](http://doc.akka.io/docs/akka/2.4/java/cluster-usage.html#cluster-usage-java) 和 [*Scala*](http://doc.akka.io/docs/akka/2.4/scala/cluster-usage.html#cluster-usage-scala) 对应章节查看集群支持部分的概述。// TODO，替换翻译链接

### 持久化

Actor 的状态变化经历可以根据需要进行持久化，或在启动或重启时进行回放。 这支持 Actor 可以恢复他们的状态，即使是在 JVM 崩溃或迁移到其他节点后。

可以在对应章节查看更多细节：[*Java*](http://doc.akka.io/docs/akka/2.4/java/persistence.html#persistence-java) 或 [*Scala*](http://doc.akka.io/docs/akka/2.4/scala/persistence.html#persistence-scala)。// TODO，替换翻译链接

### Scala 和 Java API

Akka 同时提供 [*Scala 文档*](http://doc.akka.io/docs/akka/2.4/scala.html#scala-api) and a [*Java 文档*](http://doc.akka.io/docs/akka/2.4/java.html#java-api)。

### Akka 可以以不同的方式使用

Akka 是一个工具集，而不是一个框架：你可以像集成其他库一样引入而不需要遵从特殊的代码层布局(比如 Spring)。当你以 Actor 协同的方式来表示系统时可能会感觉到被推动着对内部状态进行合适的封装，你会发现业务逻辑与内部组件通信间拥有很自然的分离。

Akka 应用通常以如下方式部署：

- 作为一个库(library)：用于 classpath 或 web 应用中的常规 JAR 包。
- 使用 [sbt-native-packager](https://github.com/sbt/sbt-native-packager) 打包。
- 使用 [Lightbend ConductR](http://www.lightbend.com/products/conductr) 打包、部署。

### 商业支持

Akka is available from Lightbend Inc. under a commercial license which includes development or production support, read more [here](http://www.lightbend.com/how/subscription).