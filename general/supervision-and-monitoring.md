# 监管和监控

本章概括了“监管”背后的概念，所提供的原语与语义。对于如何将这些转化为实际代码的相关细节，可以参考对应章节和 Scala 或 Java 的 API。

## 监管的意义

像之前 [*Actor Systems*](http://doc.akka.io/docs/akka/2.4/general/actor-systems.html#actor-systems) 中叙述的，“监管”描述了 Actor 之间的依赖关系：监管者向下级指派任务，因此也要为他们的错误做出反应。当一个下级检测的错误时(比如抛出了一个异常)，它会将自身及其所有的下级挂起并给监管者发出一个消息来为错误发送信号。基于被监管的工作和错误的性质，监管者会从下面的选项中选择一个对应的动作：

- 恢复下级，保留其积累的内部状态；
- 重启下级，清空其积累的内部状态；
- 永久关闭该下级；
- 升级错误(将该错误上报给自己的监管则)，因此自己也将失败。

总是将一个 Actor 视作一个监管层级的一部分是很重要的，这也解释了第四种选择的存在(因为一个监管者同时也是另一个更高层监管者的下级)并暗示了前三种选择：恢复一个 Actor 将会恢复其所有下级，重启一个 Actor 也会重启其所有下级(但是请看下文更多细节)，同样地，终止一个 Actor 也会终止其所有下级。需要注意的一点是，Actor 类中`preRestart`钩子的默认行为会在重启之前关闭其所有子 Actor，当然这个钩子是可以被覆写的；该钩子被执行后会递归的关闭所有剩余的子 Actor。

每个监管者会通过一个函数来配置将所有可能的错误原因转化为上面四种选项的其中之一；尤其需要说明的是，该函数并不会携带错误函数的标识符作为输入。这很容易就能想到这种看起来不太灵活的例子结构，比如，希望为不同的下级设置不同的监管策略。基于这一点，能够理解所谓监管就是要构成一个循环的错误处理结构是至关重要的。如果你尝试在同一层做的太多将会难于表达，因此推荐的方式是添加一个新的中间监管级别。

Akka 实现了一种称为”父级监管(parental-supervision)“的特殊形式。一个 Actor 只能被另一个 Actor 创建，而最顶层的 Actor 由库来提供，同时被创建的 Actor 尤其父 Actor 监管，即创建它的 Actor。这种约束使监管层级的构造更含蓄也更能促进正确的设计决策。需要注意的是这同时也限制了 Actor 不能成为孤儿或附属到外部的监管者上。另外，这也为子树形式的 Actor 应用生成了一种自然而且整洁的关闭过程。

> **警告：**
>
> 监管相关的父子间通信由特殊的系统消息进行，并且拥有与用户消息不同的邮箱。这样监管相关的事件就不会因为普通消息产生不确定性的顺序(比如普通消息消费完才能消费系统消息)。通常情况下，用户并不会受到正常消息和错误提醒的顺序影响。更多细节和示例请查看 [*Discussion: Message Ordering*](http://doc.akka.io/docs/akka/2.4/general/message-delivery-reliability.html#message-ordering) 章节。

## 顶级监管者

![akka](/assets/guardians.png)

一个 Actor 系统会在创建期间启动最少三个 Actor，如上图所示。关于 Actor 路径重要性的更多信息查看 [*Top-Level Scopes for Actor Paths*](http://doc.akka.io/docs/akka/2.4/general/addressing.html#toplevel-paths)。

### `/user`：守护 Actor

这个可能是交互最多的 Actor，作为所有用户所创建 Actor 的父 Actor。所有使用`system.actorOf()`创建的 Actor 均为名为`/user` 的 Actor 的子 Actor。这意味着一旦这个守护者终止了，所有系统中的普通 Actor 也都会被关闭。同时也意味着该守护者的监管策略决定了所有顶层的普通 Actor 如何被监管。从 Akka 2.11 开始可以使用`akka.actor.guardian-supervisor-strategy`来配置这个设置，接收一个完全限定的`SupervisorStrategyConfigurator`类名。当该守护者升级一个错误时，根(root)守护者的响应会关闭该守护者，这将导致整个 Actor 系统都会被关闭。

### `/system`：系统守护者

这个特殊的 Actor 已经介绍过了，它是为了在所有普通 Actor 终止时能够实现有序关机日志的活跃，尽管日志本身也是由 Actor 实现的。这是通过让系统守护者监控用户守护者并在接收到`Terminated`消息之后发起其自身的关闭来实现的。顶层系统 Actor 也会由一种策略监管，该策略会对所有类型的`Execption`执行无限制的重启，除了`ActorInitializationException`和`ActorKilledException`。而其他所有的可抛出异常则会被升级，导致关闭整个 Actor 系统。

### `/`：Root 守护者

Root 守护者是所有称为”顶层“ Actor 的祖父，并使用`SupervisorStrategy.stoppingStrategy`策略监管所有在 [*Top-Level Scopes for Actor Paths*](http://doc.akka.io/docs/akka/2.4/general/addressing.html#toplevel-paths)  中提到的特殊 Actor，其作用在于基于所有类型的`Exception`来终止 Actor。那其他所有的可抛出异常都会被升级？但是升级给谁呢？由于所有真实的 Actor 都有一个监管者，但 root 守护者不能是一个真实的 Actor。同时因为这意味着”outside of the bubble“，被称为”bubble-walker“(???)。这里有一个人造的`ActorRef`，它实际上会基于第一个错误符号关闭其子 Actor 并在 root 守护者完全关闭后(所有子 Actor 都被递归关闭)将系统的`isTerminated`状态设置为`true`。

## 重启的意义

当表示一个因处理一个确定的消息而失败的 Actor 时，失败的原因可以归为三类：

- 接收到特定消息后的系统性(比如程序中的)错误
- 处理消息时所使用外部资源的短暂错误
- Actor 内部状态恶化

除非错误可以被明确识别，否则第三种原因是无法被排除的，从而得出内部状态需要被清除的结论。如果监管者判断出它的其他子 Actor 或自身并未被该恶化影响，比如，应用了错误内核模式(error kernel pattern)的有意识的应用——因此最好的方式是重启子 Actor。这样就会基于 Actor 类创建一个新的实例并在 ActorRef 内部使用新的实例替换失败的实例；能够这么做的原因是因为使用了特殊的引用来封装 Actor，即 ActorRef。新的 Actor 实例会恢复处理它的邮箱，这意味着重启对 Actor 本身之外是不可见的，导致异常的这条消息也不会被重新处理。

重启期间的精确时间顺序如下：

- 挂起 Actor(在恢复之前不能再处理任何普通消息)，递归挂起所有子 Actor
- 调用旧实例的`preRestart`钩子(默认会像所有子 Actor 发送终止请求，然后调用 `poststop`)
- 在`preRestart`期间等待所有被发送终止请求的子 Actor(调用`context.stop()`)真正关闭；所有这些操作都是无阻塞的，最后一个被终止的 Actor 发出的终止提醒将会促使进入下一步
- 适应最初提供的工厂创建一个新的 Actor 实例
- 在新的 Actor 实例中调用`postRestart`(默认同时会调用`preStart`)
- 向所有在第三步中被关闭的子 Actor 发送重启请求；被重启的 Actor 同样会从 第二步开始遵循同样的递归过程
- 恢复 Actor

## 生命周期监控的意义

> **注意：**
>
> 生命周期监控在 Akka 中通常被引用为`DeatchWatch`

相对于上面描述的父子 Actor 间特殊的关系，每个 Actor 都可以监控任何其他 Actor。因为 Actor 自创建后一直是存活的，并且重启对于对受影响的监管者之外并不可见，能监控的唯一状态变化就是从存活到死亡。因此监控用于连接两个 Actor，以便在一个死亡之后另一个能做出反应，而不同于监管时用于响应错误。

生命周期监控通过一个监控 Actor 接收到`Terminated`消息来实现，如果没有以其他方式处理的话默认将会抛出一个特殊的`DeathPactException`。为了能够开始接收这个`Terminated`消息，需要调用`ActorContext.watch(targetActorRef)`。停止监听则需要调用`ActorContext.unwatch(targetActorRef)`。一个重要的特性是该消息将会不顾监控请求和目标发生终止的顺序而抵达，比如，在登记的同时仍然会收到一个消息，即便该目标已经死亡了。

当监管者不能够简单的重启子 Actor 并不得不终止他们时，监控会显得尤其有用。比如，在 Actor 的初始化阶段发生了错误。这种情况下需要监控这些子 Actor 并重新创建，或者调度自身在一段时间后重新尝试创建。

另一个常见的场景是当缺少一个外部资源时 Actor 不得不失败，该资源甚至可能是它的一个子 Actor。如果第三方通过`system.stop(child)`方法或发送一个`PoisonPill`终止了一个子 Actor，监管者则能很好的感受到。

### 延迟重启与 BackoffSupervisor 模式

`akka.pattern.BackoffSupervisor`实现了“*指数退避监管策略(exponential backoff supervision strategy)*”作为一个内建模式，它会在一个子 Actor 失败时重新启动一个，每次都会重启之间增加一个时间延时。

当被启动的 Actor 因为外部资源不可见而失败*[1]*时，这种模式非常适用，我们需要给他一些时间来再次启动。一个初级的例子是，当一个`PersistentActor`因为一个持久化错误失败时(被关闭)，可能表示数据库过载或宕机了，这种场景下更有意义的是在启动持久化 Actor 之前为数据库提供足够的时间进行恢复。

> **[1]：**一个错误可以表示两种形式：关闭或崩溃。

下面的 Scala 代码片段展示了如何创建一个退避监管者，它会在 echo Actor 因为错误而被关闭后尝试重新启动，并且每次递增一个间隔，依次是 3，6，12，24，最终称为 30 秒：

```

```

