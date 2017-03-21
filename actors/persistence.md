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

## 事件溯源

[Event Sourcing](http://martinfowler.com/eaaDev/EventSourcing.html) 背后的想法其实十分简单。一个持久化 Actor 接收一个首先被验证是否能作用于当前状态的(非持久化)消息。这里的持久化可以有很多含义，比如从简单的对命令消息的字段进行检查，到与一些外部服务的会话。如果验证成功，则会从命令生成事件，来表示该命令的副作用。这些事件会被持久化，在持久化成功之后又用于改变 Actor 的状态。当这个持久化 Actor 需要被恢复时，仅会回放那些被持久化的事件，即我们所知道的那些能成功应用的事件。换句话说，回放到一个持久化 Actor 的事件相对于命令来说是不能失败的。事件溯源 Actor 当然也可以处理那些不会改变应用状态的命令，比如一些用于查询的命令。

Akka 持久化通过`PersistentActor`特质来支持事件溯源。扩展了该特质的 Actor 通过`persist`方法来持久化和处理事件。`PersistentActor`的行为是通过实现`receiveRecover`和`receiveCommand`来定义的。这些在下面的例子中进行了演示：

```scala
import akka.actor
import akka.persistence._

case class Cmd(data:String)
case class Evt(data:String)

case class ExamplesState(events:List[String] = Nil) {
  def updated(evt:Event):ExampleState = copy(evt.data :: events)
  def size:Int = events.length
  override def toString:String = events.reverse.toString
}

class ExamplePersistenceActor extends PersistentActor {
  override def persistenceId = "sample-id-1"
  
  var state = ExampleState()
  
  def updateState(event:Evt):Unit = 
    state = state.updated(event)
    
  def numEvents = state.size
  
  val receiveRecover:Receive = {
    case evt:Evt => updateState(evt)
    case SnapshotOffer(_, snapshot:ExampleState) => state = snapshot
  }
  
  val receiveCommand:Receive = {
    case Cmd(data) =>
      persist(Evt(s"${data}-{numEvents}"))(updateState)
      persist(Evt(s"${data}-${numEvents + 1}")) { event =>
        updateState(event)
        context.system.eventStream.publish(event)
      }
    case "snap" => saveSnapshot(state)
    case "print" => println(state)
  }
}
```

例子中定义了两种数据类型，`Cmd` 和 `Evt` 分别用于表示命令和事件。`ExamplePersistentActor`的状态是一个由包含在`ExampleState`之内的持久化事件数据列表。

持久化 Actor 的 `receiveRecover` 方法定义了在恢复期间是如何通过处理`Evt`和`SnapshotOffer`消息来更新`state`的。持久化 Actor 的`receiveCommand`方法是一个命令处理器。在本例中，一个命令被处理为生成两个事件然后进行持久化和处理。通过调用`persist`方法，并传入一个事件(或一个事件列表)作为第一个参数，传入一个事件处理器作为第二个参数，将事件持久化。

`persist`方法以异步的方式持久化事件，事件处理器仅会处理那些持久化成功的消息。持久化成功的事件会为单独的消息以内部的方式发回给持久化 Actor 以触发事件处理器的执行。一个事件处理器可能会关闭或改变 Actor 的状态。已持久事件的发送者会与对应命令的发送者相同。这样就运行时间处理器会命令的发送者进行回复。

一个事件处理器的主要职责是通过事件数据来改变持久化 Actor 的状态，并通过发布新的事件来提醒其它 Actor 这些状态的成功改变。

当使用`persist`持久化事件时，持久化 Actor 会保证在`persist`调用和关联的事件处理器的执行期间不会接收到更多的命令。这也适用于在上下文中对于单个命令的多次`persist`调用。传入的消息会一直被 [*暂存(stashed)*](http://doc.akka.io/docs/akka/2.4/scala/persistence.html#internal-stash-scala) 直到`persist`结束。

如果一个事件的持久化失败了，则`onPersistFailure`会被调用，默认是将错误记录日志，同时 Actor 也会被无条件的停止(stop)。如果一个事件的持久化在保存之前被拒绝，比如因为序列化错误，`onPersistRejected`则会被调用(默认日志记录一个警告)然后 Actor 继续处理下一条消息。

运行本实例最简单的方式是下载 [Lightbend Activator](http://www.lightbend.com/platform/getstarted) 并打开名为 [Akka Persistence Samples with Scala](http://www.lightbend.com/activator/template/akka-sample-persistence-scala) 的教程。其中包含了运行`PersistentActorExample`的说明。

> **注意：**
>
> 同样可以通过`context.become()` 和 `context.unbecome()` 在正常的处理期间、恢复期间在不同的命令处理器之间进行切换。想要使 Actor 在恢复之后进入相同的状态，你需要特别小心的在`receiveRecover`方法中和`become`和`unbecome`一起执行相同的状态转换，就像在命令处理器中所做的一样。注意在回放事件的时候，从`receiveRecover`使用`become`后仍会使用`receiveRecover`的行为，在回放完成之后则会使用新的行为。

### 标识符

每个持久化 Actor 必须拥有一个标识符，并且在不同的 Actor 化身之间不能改变。这个标识符必须通过`persistenctId`方法定义。

```scala
override def persistencId = "my-stable-persistence-id"
```

> **注意**
>
> 对于在日志中提供的实体来说(数据库表/键空间)，`persistenceId`必须是唯一的。每当回放那些持久化到日志的消息时，都会通过`persistenceId`来查询消息。因此，如果不同的实体共享相同的`persistenceId`，消息回放行为则会被破坏。

### 恢复

TODO