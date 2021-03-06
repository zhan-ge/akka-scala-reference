# 为什么使用 Akka？

## Akka 平台提供了那些特性

Akka 提供了可扩展的实时事务处理。

Akka 是以下特性的一个统一运行时和编程模型：

- 纵向扩展(并发性)
- 横向扩展(远程处理)
- 容错

仅有一件事需要学习和管理，拥有很高的内聚力和连贯的语义。

Akka 是软件中一个非常可扩展的部分，不仅是性能方面，还包括使用它的应用的规模。Akka 的核心 akka-actor 非常小，很易于以无痛方式植入到那些已存在的需要异步化、无锁化(lockless)并发的项目中。

你可以选择仅引入你需要的那些部分到项目中。随着 CPU 在每个更新周期中增加越来越多的核，即便你仅使用一台机器，Akka 也可以作为一种方案来提供更加卓越的性能。Akka 还提供了一系列并发模式，支持用户为不同的工作选择合适的工具。

## Akka 的适用场景

我们了解到 Akka 在很多大型机构用于大范围作业：

- 投资与商业银行
- 零售
- 社交媒体
- 仿真模拟
- 游戏与博彩
- 汽车与交通系统
- 医疗卫生
- 数据分析

等等很多。Akka 对于任何需要高吞吐低延迟的系统来说都是一个很好的选择。

Actor 允许你管理服务错误(Supervisors)，负载管理(back-off 策略，超时和处理隔离)，同时支持纵向、横向扩展(添加更多核心或机器)。

这里有一些 Akka 用户的使用感受：

[http://stackoverflow.com/questions/4493001/good-use-case-for-akka](http://stackoverflow.com/questions/4493001/good-use-case-for-akka)

所有这些都在一个基于  ApacheV2 许可的开源项目。

