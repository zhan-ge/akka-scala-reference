# 外部贡献

该子项目为那些由外部开发者贡献的模块提供了一个主目录，随着时间的推移，可能会或许也不会移入到正式支持。这种过渡可能发生的条件包括：

- 模块中必须拥有足够的重要性以作为被采纳到正式发布的理由
- 该模块必须被积极维护
- 代码质量足够高，以支持 Akka 核心开发团队来进行高效的维护

## 用户须知

该子项目中的模块并不需要遵守在每个微型释放之间保持二进制兼容的规则。我们会基于你们的反馈进行精炼和简化，并可能在每次微型释放中引入破坏性的 API，同时也不会有任何提醒。一个实验形式的模块也可能在一个微型释放中被移除，即便之前没有被反对过。Lightbend (商业)订阅并不包括这些模块的支持。

## 当前模块列表

- [Reliable Proxy Pattern](http://doc.akka.io/docs/akka/2.4/contrib/reliable-proxy.html)
- [Throttling Actor Messages](http://doc.akka.io/docs/akka/2.4/contrib/throttle.html)
- [Java Logging (JUL)](http://doc.akka.io/docs/akka/2.4/contrib/jul.html)
- [Mailbox with Explicit Acknowledgement](http://doc.akka.io/docs/akka/2.4/contrib/peek-mailbox.html)
- [Aggregator Pattern](http://doc.akka.io/docs/akka/2.4/contrib/aggregator.html)
- [接收流水线模式](experimental-modules/external-contributions/receive-pipeline-pattern.md)
- [Circuit-Breaker Actor](http://doc.akka.io/docs/akka/2.4/contrib/circuitbreaker.html)

## 使用这些贡献的建议

因为 Akka 团队并不会对这个子项目的升级进行限制，尽管这期间会有二进制兼容的版本发布，同时模块也可能在没有反对的情况下被移除，建议直接将这些源文件复制到你的项目代码中并修改包名。这样你可以选择何时更新、需要包括哪些修复(如果需要保持二进制兼容的话)，从而后续可能发布的 Akka 版本也不会破坏你的应用。

## 共现的建议格式

每个共享都需要是一个自包含的单元，由一个源文件组成或者一个专用的包，不能依赖于该子项目中的其他模块；可以基于 Akka 发布版中的其他任何模块。这样可以保证这些贡献能够单独的移入标准发布版。该模块也必须在一个`akka.contrib`的子包中。

每个模块必须附带一个测试套件以验证所提供的功能，通常带有补充的集成测试和单元测试。测试必须遵循 [*Developer Guidelines*](http://doc.akka.io/docs/akka/2.4/dev/developer-guidelines.html#developer-guidelines) 并包含在`src/test/scala` 或 `src/test/java`目录中(与所测试的模块名匹配)。比如一个模块名为`akka.contrib.pattern.ReliableProxy`，则测试套件可以命名为`akka.contrib.pattern.ReliableProxySpec`。

同时每个模块必须拥有合适的以 [reStructured Text](http://sphinx.pocoo.org/rest.html) 格式编写的文档。该文档需要是一个单独的 `<module>.rst` 文件并处于`akka-contrib/docs`目录中，并在`index.rst`中包含一个该文档的链接。