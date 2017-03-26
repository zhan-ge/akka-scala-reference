# 集群分片

集群分片可以用于，当你想要 Actor 分布到集群的多个节点上，并通过它们的逻辑标识符来与它们交互，而不需要关心它们在集群中的物理位置，而这些物理位置也会随着时间被改变(迁移)。

比如 Actor 可以用于表示 *DDD(Domain-Driven Design)* 术语中的 *聚合根(Aggregate Roots)*。这时我们称这些 Actor 为“实体”。这些 Actor 通常拥有(坚固的)持久化状态，但是该特性并不局限于拥有持久化状态的 Actor。

集群分片通常用于有拥有很多有状态的 Actor，它们更适合一起消耗更多的资源而不是在单个机器上。如果你仅仅拥有很少几个有状态的 Actor，则让它们运行在 [集群单例](http://doc.akka.io/docs/akka/2.4/scala/cluster-singleton.html#cluster-singleton-scala) 节点上则更为简便。

在这里分片表示带有标识符的 Actor，因此成为实体，可以被自动分发到集群中的多个节点上。每个实体 Actor 仅会在一个地方运行，并且可以在发送者不知道目的 Actor 位置的情况下向其发送消息。这是通过一个由扩展提供的`ShardRegion`发送消息来实现的，它知道如何通过实体 ID 将消息路由到最终的目的地。

如果开启了对应特性，集群中那些 [*WeaklyUp*](http://doc.akka.io/docs/akka/2.4/scala/cluster-usage.html#weakly-up-scala) 状态的成员将不会激活集群分片。

> **警告**
>
> 集群分片不能和 **Automatic Downing** 一起使用，因为它会将集群拆分成两个单独的集群，从而导致在每个单独的集群中多个分片和实体被启动。查看 [*Downing*](http://doc.akka.io/docs/akka/2.4/java/cluster-usage.html#automatic-vs-manual-downing-java)。

## 一个例子

实体 Actor 看起来会是这样：

```scala
case object Increment
case object Decrement
final case class Get(counterid: Long)
final case class EntityEnvelope(id:Long, payload:Any)

case object Stop
final case class CounterChanged(delta:Int)

class Counter extends PersistentActor {
  import ShardRegion.Passivate
  
  context.setReceiveTimeout(120.seconds)
  
  // self.path.name is the entity identifier (utf-8 URL-encoded)
  override def persistenceId:String = "Counter-" + self.name
  
  var count = 0
  
  def updateState(event:CounterChanged):Unit = 
    count += event.delta
    
  override def receiveCommand:Receive = {
    case Increment => persist(CounterChanged(+1))(updateState)
    case Decrement => persist(CounterChanged(-1))(updateState)
    case Get(_) => sender ! count
    case ReceiveTimeout = context.parent ! Passivate(stopMessage = Stop)
    case Stop => context.stop(self)
  }
}
```

上面的 Actor 使用了`PersistentActor`来保存状态来实现事件溯源。它并非必须是一个持久化 Actor，但是对于处理失败和节点间的实体迁移，如果它是有价值的则必须能够恢复其状态。

注意`persistenceId`是如何被定义的。Actor 的名字在这里是标识符(utf-8 URL-encoded)。或者你可以以其他的方式来定义它，但必须是唯一的。

在使用分片扩展之前你首先应该通过`ClusterSharding.start`方法对需要支持的实体类型进行注册，通常是在集群中每个节点的系统启动的时候。`ClusterSharding.start`会给你一个可以用来传递的引用。

```scala
val counterRegion:ActoRef = ClusterSharding(system).start(
  typeName = "Counter",
  entityProps = Props[Counter],
  settings = ClusterShardingSettings(system),
  extractEntityId = extractEntityId,
  extractShardId = extractShardId
)
```

`extractEntityId`和`extractShardId`是两个由应用程序指定的函数，用于从传入的消息中解析实体标示符和分片标识符。

```scala
val extractEntityId:ShardRegin.ExtractEntityId = {
  case EntityEnvelpoe(id, payload) => (id.tostring, payload)
  case msg @ Get(id) => (id.toString, msg)
}

val numberOfShards = 100

val extractShardId: ShardRegion.ExtractShardId = {
  case EntityEnvelope(id, _) => (id % numberOfShards).toString
  case Get(id) => (id % numberOfShards).toString
}
```

这个例子展示了在消息中定义标示符的两种方式：

- `Get`消息本身已经包含了标示符
- 自定义一个`EntityEnvelope`专门用于持有标示符，实际要发送的消息则被包装在其中

注意上面的`extractEntityId`函数是如何处理这两种类型的消息的。`extractEntityId`会返回一个元组，元组的第二部分则是发送给实体 Actor 的消息，这样一来也可以根据需要将信封解包。

一个分片是一个被一起管理的实体组，而分组则是通过上面展示的`extractShardId`函数定义的。对于一个特定的实体标识符，其分片标识符总是会相同的。

创建一个好的分片算法其本身就是一个很有意思的挑战。尝试生成一个均匀的分布，比如，使每个分片的实体总数相同。作为一个经验法则，分片的数量应该是已规划的节点最大数量的十倍以上(???)。分片数量小于节点数量将会导致一些节点不会持有任何分片，而太多的分片将会导致分片的管理变的低效，比如，再平衡(rebalance)的开销，因为协调器(coordinator)会涉及到每个分片第一条消息的路由，从而增加延迟。分片算法应该在一个运行中的每个节点上保持一致。可以在集群中的所有节点关闭后进行修改。

有个简单的分片算法可以适用于大多数场景，取实体标识符的哈希绝对值并用分片数量取模(`math.abs(id.hashCode()) % shardNum`)。为了方便，已经由`ShardRegion.HashCodeMessageExtractor`提供。

给实体的消息总是通过本地的`ShardRegion`发送。`ShardRegion` Actor 作为一个被命名的实体类型的引用，由`ClusterSharding.start`返回，或者通过`ClusterSharding.shardRegion`进行检索。如果`ShardRegion`不知道一个实体的分片位置，则会进行查找。它会将消息指派到正确的节点并按需创建实体 Actor，比如，当指定实体的第一个消息到达时进行创建。

```scala
val conterRegion:ActorRef = ClusterSharding(system).shardRegion("Counter")
counterRegion ! Get(123)
expectMsg(0)

counterRegion ! EntityEnvelope(123, Increment)
counterRegion ! Get(123)
expectMsg(1)
```

一个更全面的例子可以在 [Lightbend Activator](http://www.lightbend.com/platform/getstarted) 教程中找到，名为 [Akka Cluster Sharding with Scala!](http://www.lightbend.com/activator/template/akka-cluster-sharding-scala)。

