# 事件溯源

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

默认情况下，一个持久化 Actor 会在启动和重启后通过回放被记录的日志来进行自动恢复。在回放期间发送给持久化 Actor 的消息并不会干扰这些被回放的消息。它们会被暂时缓存并在恢复阶段完成后重新被持久化 Actor 接收到。

> **注意**
>
> 访问`sender()`来回复消息总是会得到一个`deadLetters`的结果，因为原来的发送者会被认为早已消失。如果你需要在将来的恢复期间向原来的 Actor 发出一些提醒，可以在被持久化的事件中保存它的`ActorPath`。

#### 自定义恢复

应用可以在`PersistentActor`的`recovery`方法中返回一个`Recovery`对象来自定义恢复的执行方式。

想要跳过快照并重放所有事件，可以使用`SnapshotSelectionCriteria.None`。当快照的序列化格式已经被一个不兼容的方式被改变时，这种方式将会很有用。而当事件已经被删除时则不会使用这种方式：

```scala
override def recovery = Recovery(fromSnapshot = SnapshotSelectionCriteria.None)
```

另一个例子中，这可能仅是个有趣的实验而不会在实际的应用中，可以对重放设置一个上限，让 Actor 可以回放到过去的某个时间点而不是回放到其最新的状态。注意在这之后再持久化新的消息将会是个坏主意，因为跟随我们之前跳过的事件之后的这些新事件会使后续的恢复感到迷惑。

```scala
override def recovery = Recovery(toSequenceNr = 457L)
```

让`PersistentActor`的`recovery`方法返回一个`Recovery.node()`则会禁用恢复功能。

```scala
override def recovery = Recovery.none
```

#### 恢复状态

持久化 Actor 可以通过如下方法来查询自身的恢复状态：

```scala
def recoveryRunning:Boolean
def recoveryFinished:Boolean
```

有些时候需要在恢复完成并在开始处理发送到持久化 Actor 的新消息之前执行一些额外的初始化操作。持久化 Actor 会在恢复完成并在收到任何其他消息之前收到一个`RecoveryCompleted`消息。

```scala
override def receiverRecover:Receive = {
  case RecoveryCompleted =>
    // perform init after recovery, before any other messages
    // ...
  case evt => // ...
}

override def receiveCommand:Receive = {
  case msg => // ...
}
```

Actor 总是会收到一个`RecoveryCompleted`消息，甚至当日志记录中没有消息或者快照存储为空的时候，或者一个带有之前从未被用过的`persistenceId`的新持久化 Actor。

当一个 Actor 在从日志恢复状态时出现了问题，`onRecoveryFailure`会被调用(默认是将错误记录到日志)，同时 Actor 会被停止(stop)。

### 内部暂存(stash)

持久化 Actor 拥有一个私有的  [*stash*](http://doc.akka.io/docs/akka/2.4/scala/actors.html#stash-scala)，用于正在恢复时或`persist\persistAll`方法正在持久化事件时来内部的缓存进入的消息。你仍然可以使用或继承`Stash`接口。内部的 stash 通过勾住(hook-into)`unstashAll`来与普通的 stash 进行配合，并同时确保消息会被正确的释放给内部的 stash 来维持顺序保证。

你要当心不能发送超出了其所能承受范围的消息数量给一个持久化 Actor，否则被暂存消息的数量将持续没有上限的上涨。可以在邮箱配置中设置最大暂存容量来避免`OutOfMemoryError`：

```scala
akka.actor.default-mailbox.stash-capacity=10000
```

注意这个暂存容量针对的是单个 Actor。如果你拥有大量持久化 Actor，当使用集群分片的时候，你可能需要定义一个小的暂存容量来确保系统中被暂存消息的整体数量不会消耗太多内存。另外，持久化 Actor 定义了三种策略来处理内部暂存容量溢出的错误。默认的溢出策略是`ThrowOverflowExceptionStrategy`，它会抛弃当前接收到的消息并抛出一个`StashOverflowException`异常，如果使用默认的监管策略则会引起 Actor 重启。你可以为任何"个别的"持久化 Actor 覆写`internalStashOverflowStrategy`方法来返回一个`DiscardToDeadLetterStrategy`或`ReplyToStrategy`，或者提供 FQCN 来为所有的持久化 Actor 提供默认设置，但必须是`StashOverflowStrategyConfigurator`的子类，最后在持久化配置中进行设置：

```scala
akka.persistence.internal-stash-overflow-strategy = 
  "akka.persistence.ThrowExeceptionConfigurator"
```

这个`DiscardToDeadLetterStrategy`策略同时拥有一个配套的预打包配置器`akka.persistence.DiscardConfigurator`。

你同样可以通过 Akka 持久化扩展单例来查询默认策略：

```scala
Persistence(context.system).defaultInternalStashOverflowStrategy
```

> **注意**
>
> 持久化 Actor 中应该避免有界邮箱，因为来自存储后端的消息可能会被丢弃。你可以使用有界暂存容量来替换有界邮箱。

### 不严格的本地一致性要求和高吞吐场景

如果要面对不严格的本地一致性和高吞吐要求，有些时候`PersistenceActor`和它的`persist`方在消费高速传入的命令时可能会显得不够用，因为他要等到所有与命令相关的事件都被处理完以处理下一条命令。虽然这种抽象适用于大多数场景，有些时候你会需要面对一致性要求——比如你需要以最快的速度处理命令，同时假设事件在后台最终能够被持久化并处理成功，如果需要的话再进行回溯来响应持久化错误。

`persistAsync`方法提供了一个工具来实现高吞吐的持久化 Actor。当日志正在执行持久化或用户代码正在执行事件回调时它不会暂存传入的命令。

在下面的例子中，事件回调可能会在“任何时间”进行，甚至在下一条命令被处理完之后。而事件之间的顺序仍然是有保证的("evt-b-1" 仍然会在 “evt-a-2” 之后被发送等等)。

```scala
class MyPersistentActor extends PersistentActor {
  override def persistenceId = "my-stable-persistence-id"
  
  override def receiveRecover:Receive = {
    case _ => // handle recovery here
  }
  
  override def recieveCommand:Receive = {
    case c:String =>{
      sender() ! c
      persistAsync(s"evt-$c-1") {e => sneder() ! e}
      persistAsync(s"evt-$c-2") {e => sender() ! e}
    }
  }
}

// usage
persistentActor ! "a"
persistentActor ! "b"

// possible order of received messgaes:
// a
// b
// evt-a-1
// evt-a-2
// evt-b-1
// evt-b-2
```

> **注意**
>
> 为了能够实现“事件溯源(*command sourcing*)”模式，可以简单的对所有传入的消息立即调用`persistAsync(cmd)(...)`并在回调中处理它们。

> **警告**
>
> 在`persistAsync`调用和日志确认写入结束之间如果 Actor 被重启或关闭，则回调将不会被调用。

### 延迟操作直到前一个持久处理器执行完成

当你使用`persistAsync`的有些时候可能会发现，定义一些“延迟到前一个持久处理器执行完成“才会执行的操作会很有用。因此`PersistentActor`提供了一个名为`deferAsync`的方法，它与`persistAsync`的工作方式类似，但是并不会持久化传入的事件，仅让定义的动作延迟执行。它被推荐用于”读“操作，以及在你的领域模型中没有对应事件的操作。

该方法的使用方式与持久类的方法类似，只是它并不会持久化传入的事件。仅会将事件保持在内存中并在处理器被调用的时候使用。

```scala
class MyPersistentActor extends PersistentActor {
  override def persistenceId = "my-stable-persistence-id"
  
  override def receiveRecover:Receive = {
    case _ => // handle recovery here
  }
  
  override def receiveCommand:Receive = {
    case c:String =>{
      sender() ! c
      persistAsync(s"evt-$c-1") {e => sender() ! e}
      persistAsync(s"evt-$c-2") { e => sender() ! e }
      deferAsync(s"evt-$c-3") { e => sender() ! e }
    }
  }
}
```

注意，在回调处理器中访问`sender()`是安全的，它会指向原始的命令发送者，即该`deferAsync`处理器的调用者。

调用方会得到如下(保证)有序的响应：

```
persistentActor ! "a"
persistentActor ! "b"
 
// order of received messages:
// a
// b
// evt-a-1
// evt-a-2
// evt-a-3
// evt-b-1
// evt-b-2
// evt-b-3
```

> **警告**
>
> 在`deferAsync`调用和之前的日志被处理并全部确认写入完成之间如果 Actor 被重启或关闭，则回调将不会被调用。

### 嵌套持久化调用

可以在`persist`和`persistAsync`各自的回调块中再次调用这些方法，同时仍能够正确的保证两者的线程安全性(包括正确的 sender() 值)，以及有保证的暂存顺序。

通常来说更鼓励创建那些不需要依赖嵌事件持久的命令处理器，但是有些场景中这些方法会更为有用。能够理解这种情况下的回调执行顺序是很重要的，以及它们所隐含的暂存行为(`persist()` 方法强制引起的)。下面例子中发出了两个持久调用，而每一个各自的回调块中又会发出另一个持久调用：

```scala
override def receiveCommand:Receive = {
  case c:String =>
    sender() ! c
    
    persist(s"$c-1-outer"){ outer1 =>
      sender() ! outer1
      persist(s"$c-1-inner") { inner1 =>
        sender() ! inner1
      }
    }
    
    persist(s"$c-2-outer") { outer2 =>
      sender() ! outer2
      persist(s"$c-2-inner"){ inner2 =>
        sender() ! inner2
      }
    }
}
```

当像该 Actor 发送两个命令，持久化处理器将会按如下顺序处理：

```
persistentActor ! "a"
persistentActor ! "b"
 
// order of received messages:
// a
// a-outer-1	<-
// a-outer-2	<- should be "a-inner-1"
// a-inner-1
// a-inner-2
// and only then process "b"
// b
// b-outer-1
// b-outer-2
// b-inner-1
// b-inner-2
```

首先这些外层的持久调用被发出，之后又应用了它们的回调。当这些成功完成之后，内层的回调将会被调用(一旦它们需要持久的事件被日志确认持久化之后)。只有所有这些处理器都成功完成之后下一条命令才会抵达当前的持久化 Actor。换句话说，传入消息的的暂存是由外层最开始的`persist()`来保证的，它会被延长直到所有嵌套的`persist`都被处理完。

可以使用同样的方式将`persistAsync`进行嵌套：

```scala
override def receiveCommand:Receive = {
  sender() ! c
  persistAsync(c + "-outer-1") { outer =>
    sender() ! outer
    persistAsync(c + "-inner-1"){ inner => sender() ! inner }
  }
  persistAsync(c + "-outer-2"){ outer =>
    sender() ! outer
    persistAsync(c + "-inner-2"){ inner => sender() ! inner }
  }
}
```

这种情况下则不会发生暂存，然而事件的持久化和回调的执行都能符合预期的顺序：

```
persistentActor ! "a"
persistentActor ! "b"
 
// order of received messages:
// a
// b
// a-outer-1
// a-outer-2
// b-outer-1
// b-outer-2
// a-inner-1
// a-inner-2
// b-inner-1
// b-inner-2
 
// which can be seen as the following causal relationship:
// a -> a-outer-1 -> a-outer-2 -> a-inner-1 -> a-inner-2
// b -> b-outer-1 -> b-outer-2 -> b-inner-1 -> b-inner-2
```

虽然能够嵌套混合的`persist`和`persistAsync`并保持其各自的语义，但并不推荐这么做，这会造成过于复杂的嵌套。

> **警告**
> 虽然能够在`persist`调用中嵌套另一个该调用，但是不能从消息处理的当前线程之外的任何线程来调用`persist`。比如不能在`Future`中调用`persist`！这么做会打破持久化方法最初能够提供的保证。因此，总是在 Actor 的接收块中调用`persist`或`persistAsync`(或从这里同步调用的方法)。

### 失败

如果一个事件的持久化失败了，则会调用`onPersistFailure`(默认将错误打印到日志)，同时 Actor 将会被无条件的停止。

无法恢复的原因是无法知道该事件实际上是否被持久化成功了，因此存在一个不一致的状态。基于持久化错误的重启操作通常也总是会失败，因为日志可能已经不可用了。更好的方式是关闭 Actor 并通过一个退避超时来将其重新启动。`akka.pattern.BackoffSupervisor`提供了这种方式的重启。

```scala
val childPros = Props[MyPersistentActor]
val props = BackoffSupervisor.props(
  Backoff.onStop(
    childProps,
    childName = "myActor",
    minBackoff = 3.seconds,
    maxBackoff = 30.seconds,
    randomFactor = 0.2
  )
)
context.actorOf(props, name="mySupervisor")
```

如果一个事件在完成存储之前被拒绝，比如，因为序列化错误，则会调用`onPersistRejected`(默认将警告打印到日志)，Actor 则会继续处理下一条消息。

如果当 Actor 启动时从日志恢复内部状态出现了问题，则会调用`onRecoveryFailure`(默认将错误打印到日志)，Actor 将会被停止。注意因为加载快照导致的错误也会被相似的方式处理，但是如果你知道序列化格式已经被以不兼容的方式修改了则可以禁用快照的加载，查看 [自定义恢复](http://doc.akka.io/docs/akka/2.4/scala/persistence.html#recovery-custom-scala)。

### 原子写

每个事件当然都会以原子的方式保存，同样可以使用`persist`或`persistAsync`方法将多个事件以原子的方式保存。这意味着要么所有的事件都被保存成功，要么因为一个错误而导致所有事件都没有被保存成功。

持久化 Actor 的恢复也永远不会仅回放那些通过`persistAll`保存的事件的一个子集。

有些日志可能不支持多个事件的原子写，它们会拒绝`persistAll`命令，比如，带有一个异常(通常会是 UnsupportedOperationException)的`onPersistRejected`将会被调用。

### 批量写

为了在使用`persistAsync`时优化吞吐，持久化 Actor 会在高负载的情况下在将事件写入日志之前内部的将它们批量化(作为单个批)。批的数量大小将会在日志的来回周期时间内根据发出事件的多少来动态决定：在向日志发送一个批之后并在接收到该批的写入完成确认之前不会有更多的批会被发送。批量写永远不会基于计时器，而是将延时保持在最小。

### 消息删除

能够将单个持久化 Actor 所记录的直到一个序号的所有消息删除。持久化 Actor 可以调用`deleteMessages`来实现这一点。

在基于时间溯源的应用中，消息删除要么甚至都不会使用，要么会结合[快照打印](http://doc.akka.io/docs/akka/2.4/scala/persistence.html#snapshots)一起使用，比如，当在一个快照被成功保存之后，调用`deleteMessages(toSequenceNr)`并提供一个截止数据的序号来安全地删除那些已经由快照持有的事件，然后在消息回放时仍能通过快照来访问所有已积累的状态。

> **警告**
> 如果你正在使用 [持久化查询](Persistence Query)，查询结果中将会丢失那些已在日志中删除的消息，这取决于日志插件中的删除是如何实现的。除非你使用了一个仍然会在查询结果中显示已删除消息的插件，否则你不得不设计以的应用以避免被这些丢失的消息所影响。

如果`deleteMessages`请求删除成功，则会给持久化 Actor 发送一个`DeleteMessagesSuccess`消息，失败则是`DeleteMessagesFailure`消息。

消息删除并不会被日志的最好序号影响，甚至如果因为从他开始调用`deleteMessages`会删除所有消息。

### 持久化状态处理

消息的持久化、删除和回放，要么成功要么失败。

| Method               | Success               | Failure/Rejection     | 失败处理器调用之后 |
| -------------------- | --------------------- | --------------------- | --------- |
| persist/persistAsync | 调用持久化处理器              | onPersistFailure      | Actor 被关闭 |
| persist/persistAsync | 调用持久化处理器              | onPersistRejected     | 没有自动操作    |
| recovery             | RecoveryCompleted     | onRecoveryFailure     | Actor 被关闭 |
| deleteMessages       | DeleteMessagesSuccess | DeleteMessagesFailure | 没有自动操作    |

最重要的操作，`persist`和`recovery`，拥有错误处理器并以显式回调的形式存在，并且用户能够在`PersistentActor`中覆写。这些处理器的默认实现是打印日志，将引起错误的消息及错误的原因和信息记录到日志。

对于那些决定性的失败，比如恢复或持久化事件时的失败，持久化 Actor 会在失败处理器被处理之后关闭。