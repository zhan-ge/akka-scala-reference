# 什么是 Actor

上一节的 Acotr System 解释了 Actor 如何组成层级结构并作为创建应用时的最小单元。这节将单独的观察一个 Actor，解释一些你在实现时会遇到的概念。更加深入的具体细节可参考 [*Actors (Scala)*](http://doc.akka.io/docs/akka/2.4/scala/actors.html#actors-scala)  和 [*Untyped Actors (Java)*](http://doc.akka.io/docs/akka/2.4/java/untyped-actors.html#untyped-actors-java)。

一个 Actor 是一个包含  [状态](http://doc.akka.io/docs/akka/2.4/general/actors.html#state), [行为](http://doc.akka.io/docs/akka/2.4/general/actors.html#behavior), [Mailbox](http://doc.akka.io/docs/akka/2.4/general/actors.html#mailbox), [子 Actor](http://doc.akka.io/docs/akka/2.4/general/actors.html#child-actors) 及对应 [监管策略](http://doc.akka.io/docs/akka/2.4/general/actors.html#supervisor-strategy) 的容器。所有这些都被封装在一个 [Actor 引用](http://doc.akka.io/docs/akka/2.4/general/actors.html#actor-reference) 之中。需要注意的一个方面是 Actor 拥有清晰的生命周期，在不被引用自后并不会自动销毁；在创建之后你也有责任确保其能够将其最终关闭，并且你也拥有在 [Actor 关闭时](http://doc.akka.io/docs/akka/2.4/general/actors.html#when-an-actor-terminates) 将其资源释放的控制权。

## Actor 引用

如下面的细节中所述，一个 Actor 需要与外部隔离以获得 Actor 模型的优势。因此，Actor 对外部表示为一个 Actor 引用，这些对象可以没有任何约束的任意传递。这样拆分的内部对象和外部对象使所有期望得到的操作都透明化了：重启一个 Actor 而无需再别处更新引用，将实际的 Actor 对象部署在远程主机，在完全不同的应用间发送消息。但最重要的一面是这样就不能再从外部查看 Actor 的内部并持有其状态，除非 Actor 本身愚蠢的向外发布信息。

## 状态

Actor 对象通常会持有一些变量来反映其当前所处的状态。这可以是一个显式的状态机(比如使用 [*FSM*](http://doc.akka.io/docs/akka/2.4/scala/fsm.html#fsm-scala) 模块)，或者一个计数器，或设置监听者，或暂存请求，等等。这些数据让 Actor 变得有价值，而且需要加以保护以避免被其他 Actor 污染。好消息是 Akka 中的 Actor 在概念上各自拥有自己的轻量级线程(并非实际的内核线程)并且完全屏蔽了系统的其他部分。这意味着你不再需要以锁的方式来同步访问，你只需编写 Actor 代码而无需关心并发问题。

在这背后，Akka 实际会将一组 Actor 运行在一组真实的线程上，通常情况下多个 Actor 会共享同一个线程，一个 Actor 中一系列顺序化的调用最终也会由多个不同的线程处理。Akka 确保了这种实现细节不会影响处理 Actor 状态的单线性(single-threadedness)。

因为内部状态对于 Actor 的操作来说是至关重要的，拥有不一致状态将会是致命的。因此，当一个 Acotr 失败并被其监管者重启时，状态将会被重建，就像一开始创建时一样。这种特性让系统拥有自愈能力。

或者，可以通过将收到的消息持久化并在重启后重放的方式，将 Actor 恢复到重启前的状态(查看 [*Persistence*](http://doc.akka.io/docs/akka/2.4/scala/persistence.html#persistence-scala))。

## 行为

每当一个消息被处理，它都会被匹配到 Actor 当前的行为。行为表示一个函数，它定义了在一个时间点对消息的反应所采取的行动，比如用户被授权则转发请求，否则就拒绝。这种行为会随着时间改变，比如，不同的客户端会随着时间的推移而获得授权，再或者 Actor 可能进入”停工“模式然后又恢复。要实现这种改变，要么将其编码到状态变量并由行为逻辑读取，要么在运行时将函数自身进行交换，比如`become`和`unbecome`操作。然而，Actor 对象在构造期间定义的初始行为会比较特殊，因为当 Actor 重启时会将其恢复到最初的这个行为。

## MailBox

Actor 的目的在于处理消息，这些消息则发送自其它 Actor(或从 Actor 系统外部)。用于连接发送者和接收者的这个部分便是邮箱：每个 Actor 都会有且只有一个邮箱，所有的发送者都会将消息入队到该邮箱。入队会按照发送操作的时序进行，这意味着，由于 Actor 跨线程分布这样明显的随机性，在运行时期间，从不同 Actor 发送的消息并不会拥有确定的顺序。但是，从同一个 Acotr 向另一个 Actor 发送的多个消息则会以与发送时相同的顺序入队。

有多种不同的邮箱实现可供选择，默认的方式是 FIFO：以消息入队的顺序来处理消息。这种默认的方式普遍适用，不过有些应用可能需要对消息设置优先级。在这种场景中，入队时消息不再被放到邮箱的末端，而是按照消息对应的优先级放到指定的位置，甚至会被放到最前面。使用这种队列时，消息的处理顺序将会自然的按照队列算法的设定，通常也再是 FIFO 的方式。

Akka 中实现的 Actor 模型区别于其他实现的一个重要特性是当前的行为总是必须会用来处理下一条出队的消息，并不会搜索邮箱来查找下一个匹配的消息。处理消息失败通常会当做错误对待，除非该行为被覆盖了。

## 子 Actor

每个 Actor 都是一个潜在的监管者：如果它创建了子 Actor 来指派任务，它将会自动监管起这些子 Actor。子 Actor 列表由 Actor 的`context`维护并由该 Actor 访问。对该列表的修改，比如创建可以通过`context.actorOf(...)`完成，`context.stop(child)`则用于关闭，这些操作会立即反应。实际的创建或关闭动作会在背后以异步的方式执行，因此不会阻塞监管者。

## 监管策略

Actor 的最后一部分是其处理子 Actor 错误的策略。错误处理会由 Akka 以透明的方式完成，将声明在 [*监管和监控*](http://doc.akka.io/docs/akka/2.4/general/supervision.html#supervision)  中的其中一个策略应用到传入的错误。因为该策略是构建 Actor 系统的基础，因此一旦 Actor 被创建则无法再进行修改。

有明白每个 Actor 只能有一个这样的策略，这意味着如果需要不同的策略应用到了不同的子 Actor，这些子 Actor 需要根据匹配的策略由中间监管者分组，这样就再次印证了 Actor 系统的组织方式，即将任务拆分为多个子任务。

## 一个 Actor 何时关闭

一个 Actor 一旦关闭，比如，由于一个没有被处理的错误并且没有被重启，关闭了自身或者被监管者关闭，它将会释放它的资源，将邮箱中所有的消息倾倒到系统的”死亡信箱“，这些消息则会被当做 DeadLetter 转发到 EventStream。该邮箱则会在 Actor 引用中被替换为一个系统邮箱，将所有新的消息当做 DeadLetter 转发到 EventStream。这是在尽最大努力的基础上完成的，因此不要依靠这些来实现”抵达性保证“。

这种不默默倾泻消息的方式源于我们的测试启发，我们将 TestEventListener 注册到被转发死信的事件总线，这样每次接收到死信都会打印一个警告日志，这种方式能够帮助加速错误测试。因此可以想象这个功能或许可以用于其他用途。