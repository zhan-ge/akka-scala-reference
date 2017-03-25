# 持久化

Akka 的持久化能够使有状态的 Actor 持久化其内部状态，从而在启动、因为 JVM 崩溃或监管者造成的重启、迁移到一个集群的时候进行恢复。Akka 持久化背后的关键概念是仅对那些能够改变 Actor 内部状态的改变进行持久化，而非直接持久化当前的状态(除非是可选的快照)。这些改变仅能追加到存储，一切都是不可变的，从而能够支持那些高事务率和高效率的应用。Actor 通过回放那些已保存的、可用与重建内部状态的改变来进行恢复。可以选择完整历史回放，或者从一个快照开始，这样能显著减少恢复所花费的时间。Akka 持久化同时能够提供消息最少一次抵达语义的点对点通信。

Akka 持久化的灵感来自于 [eventsourced](https://github.com/eligosource/eventsourced) 库，并且成为该库的官方替代。它遵循了与 [eventsourced](https://github.com/eligosource/eventsourced) 相同的概念和架构，但是 API 和 实现级别用于明显的区别。可以查看  [*Migration Guide Eventsourced to Akka Persistence 2.3.x*](http://doc.akka.io/docs/akka/2.4/project/migration-guide-eventsourced-2.3.x.html#migration-eventsourced-2-3)。

## 依赖

Akka 持久化是一个单独的 jar 文件。确保你的项目中已经添加了如下依赖：

```
"com.typesafe.akka" %% "akka-persistence" % "2.4.17"
```

Akka 持久化扩展带有一些内建的持久化插件，包括基于内存堆的日志，基于本地文件系统的快照存储，以及基于 LevelDB 的日志。

基于 LevelDB 的插件需要一些额外的依赖声明：

```
"org.iq80.leveldb"            % "leveldb"          % "0.7"
"org.fusesource.leveldbjni"   % "leveldbjni-all"   % "1.8"
```

## 架构

- **PersistentActor**：是一个持久化的、有状态的 Actor。它可以将事件持久化到日志并能以线程安全的方式响应这些事件。它可以被用于实现 *command* 或称为 *event sourced* 的 Actor。当一个持久化 Actor 被启动或重启时，已记录的消息会被回放到这些 Actor，从而恢复通过这些消息来恢复内部状态。
- **PersistentView**：一个持久化、有状态的 Actor 接收那些已经被另一个持久化 Actor 写入日志的消息，则前一个 Actor 被称为*视图*。一个试图并不会记录新的消息，替代的是，它仅通过另一个持久化 Actor 的复制消息流来更新内部状态。
- **AtLeastOnceDelivery**：发送那些带有最少一次抵达语义的消息给目标，同样包括发送者或接收者因为 JVM 崩溃的情况。
- **AsyncWriteJournal**：一个存储了发送给持久化 Actor 的消息序列的日志。一个应用可以控制哪些消息要被记录或者哪些被持久化 Actor 接收的消息不用记录。日志维护一个随着每条消息递增的*highestSequenceNr*，而日志背后的存储是可插拔的。持久化扩展带有一个写入到本地文件系统的 “levelDB” 插件。复制集日志则通过 [社区插件](http://akka.io/community/) 支持。
- **快照保存(Snapshot store)**：一个快照保存会持久化一个持久化 Actor 或视图的内部状态。快照用于优化恢复时间。快照保存背后的存储也是可插拔的。持久化扩展带有一个写入本地文件系统的“本地”快照存储插件。复制集快照保存则通过 [社区插件](http://akka.io/community/) 支持。

