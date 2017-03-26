# 工作原理

集群中的每个节点都会启动 `ShardRegion` Actor，或者被被标记为特定角色(role)的一组节点。`ShardRegion`由两个应用指定的函数创建，分别用于为传入的消息解析实体标识符和分片标识符。一个分片是一组被一起管理的实体。对于特定分片的第一条消息，`ShardRegion`会从中心协调器 `ShardCoordinator` 来请求其位置。

`ShardCoordinator`会决定哪个`ShardRegion`拥有对应分片并通知该`ShardRegion`。`ShardRegion`会确认请求并以子 Actor 的方式创建一个`Shard`管理员。当`Shard` Actor 需要的时候，单个的`Entities`则会被创建。传入的消息则会通过`ShardRegion`和`Shard`传递到目标`Entity`。

如果分片的位置属于另一个`ShardRegion`实例，消息则会被转发到该实例。在查找分片位置期间，传入到分片的消息会被缓存并在找到分片位置之后被送达。随后发送到这个分片的消息则直接能够抵达目标而无需再涉及到`ShardCoordinator`。

场景一：

1. 传入的消息 M1 到`ShardRegion`实例 R1
2. M1 被映射到分片 S1，R1 并不知道 S1，因此向协调器 C 请求 S1 的位置
3. C 回答了 S1 的位置在 R1
4. R1 为实体 E1 创建子 Actor 并向其发送被缓存的从 S1 到 E1 的消息
5. 所有从 S1 到达 R1的消息这时可以直接由 R1 来处理而不再经过 C。R1 会按需创建实体的子 Actor，并向它们转发消息


场景二：

1. 传入消息 M2 到 R1
2. M2 被映射到 S2，R1 并不知道 S2，因此向协调器 C 请求 S2 的位置
3. C 回答了 S2 的位置是 R2
4. R1 将那些要发送给 S2 并已被缓存的消息发送给 R2
5. 所有要发送给 S2 的并到达 R1 的消息将会通过 R1 处理，不再经过 C。R1 将消息发送给 R2
6. R2 接收到了要发送给 S2 的消息，请求 C，得到 S2 的位置在 R2，这时就回到了第一个场景

要确保一个特定的实体 Actor 在集群中最多只能有一个实例运行在集群的某个位置，这很重要，因为分片所处的集群中所有的节点要拥有相同的视图(view)。因此分片的分配决定由中心`ShardCoordinator`来完成，它作为一个集群单例运行，比如，运行在集群中所有节点或由角色名标记的一组节点中的最老节点上的实例。

用来决定分片所处位置的逻辑由一个可插拔的分片分配策略定义。默认的实现是`ShardCoordinator.LeastShardAllocationStrategy`，它会在之前已分配的分片数量最少的`ShardRegion`上分配新的分片。这个策略可以被应用指定的实现替换。

为了能够使用集群中新加入的成员，协调器会帮助将分片重新均衡，比如，将实体从一个节点迁移到另一个节点。在重新均衡的过程中，协调器首先会通知所有`ShardRegion` Actor 一个分片的切换已经开始了。这意味着它们将会开始缓存发送到这些分片的消息，就像还不知道分片位置的时候一样。在重新均衡期间，协调器不会再应答任何正在被重新均衡的分片的位置，比如，本地的缓冲将会持续到切换结束。那些负责被重新均衡的分片的`ShardRegion`会通过一个`handOffStopMessage`消息(默认为`PoisonPill`)将原有分片中的实体关闭。当所有的分片都已被终止，正在持有新实体的`ShardRegion`将会向协调器发送确认消息来完成切换。然后协调器开始回复那些获取分片位置的请求并为分片分配一个位置，被`ShardRegion` Actor 缓冲的消息也会抵达新的位置。这意味着实体的状态并不会被转换为迁移。如果实体的状态很重要，则需要把他们定义为(坚固的)持久化 Actor，比如 [*Persistence*](http://doc.akka.io/docs/akka/2.4/scala/persistence.html#persistence-scala)  ，从而能够在新的位置进行恢复。

用来决定重新均衡哪个分片的逻辑定义在一个可插拔的分配策略中。默认的实现是`ShardCoordinator.LeastShardAllocationStrategy`，它会在之前已分配数量最多的`ShardRegion`中选择分片。然后分配到那些之前已分配分片的数量最少的`ShardRegion`，比如加入到集群中的新成员。有一个可配置的阈值来决定差值有多大的时候需要开始重新均衡。这个策略可以被应用指定的实现替换。

`ShardCoordinator`中关于分片位置的状态会通过 [*Persistence*](http://doc.akka.io/docs/akka/2.4/scala/persistence.html#persistence-scala) 进行(坚固的)持久化以幸免于失败。因此它所运行的集群必须通过一个分布式的日志来配置 [*Persistence*](http://doc.akka.io/docs/akka/2.4/scala/persistence.html#persistence-scala) 。当一个崩溃或无法达到的协调器节点被从集群中移除时，新的`ShardCoordinator`单例会接管并恢复其状态。在这个失败期间，那些已知道位置的分片将仍然可用，而发送到那些新的或不知道位置的分片的消息将会被缓冲，直到新的`ShardCoordinator`变为可用。

只要一个发送者使用相同的`ShardRegion` Actor 向实体 Actor 分配消息，消息的顺序则会被保持。只要没有到达缓冲的上限，消息则会以最大努力原则抵达，即最多一次抵达的语义，和常规消息的发送方式一样。可靠的端对端消息发送，可以通过在 [*Persistence*](http://doc.akka.io/docs/akka/2.4/scala/persistence.html#persistence-scala) 中添加`AtLeastOnceDelivery`来实现最少一次抵达的语义。

消息被发送到一个新的或之前没被用过的分片时，因为和协调器的往返交互，会引入一些额外的延迟。分片的重新负载也会引入新的延迟。这在设计应用的分片方案时需要慎重考虑，比如避免粒度太小的分片。