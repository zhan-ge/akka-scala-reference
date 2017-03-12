# 实验性模块

一下这些 Akka 模块被标记为实验性质，这表示他们仍处于早期访问模式，同时这也表示他们没有商业部分的支持。以早期实验性质发布的目的在于使他们更易于可用并基于反馈来逐步改良，或者甚至发现他们并不是很有用。

一个实验性质的模块不需要遵守在每个微型释放之间保持二进制兼容的规则。我们会基于你们的反馈进行精炼和简化，并可能在每次微型释放中引入破坏性的 API，同时也不会有任何提醒。一个实验形式的模块也可能在一个微型释放中被移除，即便之前没有被反对过。

- [Multi Node Testing](http://doc.akka.io/docs/akka/2.4/dev/multi-node-testing.html)
- [Actors (Java with Lambda Support)](http://doc.akka.io/docs/akka/2.4/java/lambda-actors.html)
- [FSM (Java with Lambda Support)](http://doc.akka.io/docs/akka/2.4/java/lambda-fsm.html)
- [Persistence Query](http://doc.akka.io/docs/akka/2.4/scala/persistence-query.html)
- [Akka Typed](http://doc.akka.io/docs/akka/2.4/scala/typed.html)

将模块标记为实验性质的另一个原因是有些模块在现在还很难确定是否能找到一个能够持续负责该模块的维护者。这类模块作为子项目包含在`akka-contrib`中：

- [外部贡献](experimental-modules/external-contributions.md)

