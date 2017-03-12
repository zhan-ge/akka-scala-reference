# 接收流水线模式

该模式支持你为消息定义通用的拦截器并将配置任意数量的拦截器配置的你的 Actor 中。能够用于定义交叉切面并应用到多个或全部 Actor。

## 一些可能的用例

- 度量处理消息的时长
- 审计关联发送者的消息
- 为重要消息记录日志
- 为消息设置安全限制
- 文本国际化

## 拦截器

多个拦截器可以通过混入`ReceivePipeline`特质来添加到 Actor 中。这些拦截器会为 Acotr 的行为定义多层装饰器。第一个拦截器会定义一个外层装饰器，然后指派到对应第二个拦截器的装饰器上，等等，直到最后一个拦截器为 Actor 的`Receive`定义一个装饰器。

第一个或最外层的拦截器会接收到发生给该 Actor 的消息。

对于收到的每个消息，拦截器会基于消息执行一些处理并决定是否将该消息发送给下一个拦截器。

一个拦截器是`PartialFunction[Any, Delegation]`的类型别名。`Any`类型的输入是从上一个拦截器收到的消息(或作为第一个拦截器时直接接收自消息发送者)。返回类型`Delegation`控制了是否将消息传送给下一个拦截器。

## 一个简单的例子

为了将一个转换后的消息发送给 Actor，拦截器会返回一个`Inner(newMsg)`，而`newMsg`则是被转换后的消息。

下面的拦截器展示了这个过程。它会拦截`Int`消息，向其执行加 1 操作并将递增后的消息传递给下一个拦截器：

```scala
val incrementInterceptor:Interceptor = {
  case i: Int => Inner(i + 1)
}
```

## 构建流水线

为了让你的 Actor 拥有将接收到的消息流水线化的能力，需要混入`ReceivePipeline`特质。他有两个方法来控制流水线，`pipelineOuter`和`pipelineInner`，它们就接收一个`Interceptor`。第一个方法将拦截器添加到流水线的开头，后一个方法将拦截器添加到流水线的结尾，仅处于 Actor 的当前行为之前。

在这个例子中我们将`ReceivePipeline`特质混入 Actor 并通过`pipelineInner`添加了`Increment`和`Double`这两个拦截器。这样两个拦截器会在最后被应用：

```scala
class PipelinedActor extends Actor with ReceivePipeline{
  // Increment
  pipelineInner { case i:Int => Inner(i + 1) }
  // Double
  pipelineInner { case i:Int => Inner(i *2) }
  
  def receive:Receive = { case any => println(any) }
}

actor ! 5 // prints 12 = (5 +1) *2
```

如果我们使用`pipilineOuter`来添加`Double`，则该操作将会先于`Increment`被应用：

```scala
// Increment
pipelineInner { case i:Int => Inner(i + 1) }
// Double
pipelineOuter { case i:Int => Inner(i *2) }

// prints 11 = (5 *2) +1
```

## 拦截器混入

将所有的拦截器实现在 Actor 的内部可以很好的用于展示该模式，单不是非常实用。真正灵活的实现是将每个拦截器定义在各自的特质中，然后混入到任何你需要的 Actor 中。

让我们在一个例子中看一下。我们拥有如下模型：

```scala
val text = Map(
  "that.rug_EN" → "That rug really tied the room together.",
  "your.opinion_EN" → "Yeah, well, you know, that's just, like, your opinion, man.",
  "that.rug_ES" → "Esa alfombra realmente completaba la sala.",
  "your.opinion_ES" → "Sí, bueno, ya sabes, eso es solo, como, tu opinion, amigo."
)

case class I18nText(local: String, key: String)
case class Message(author:Option[String], text:Any)
```

然后是两个拦截器的定义，每个都在其各自的特质中实现：

```scala
trait I18nInterceptor {
  this: ReceivePipeline =>
  
  pipelineInner {
    case m @ Message(_, I18nText(loc, key)) =>
      Inner(m.copy(text = texts(s"${key}_$loc")))
  }
}

trait AuditInterceptor {
  this: ReceivePipeline =>
  
  pipelineOuter {
    case m @ Message(Some(author), text) =>
      println(s"$author is about to say: $text")
      inner(m)
  }
}
```

第一个拦截器拦截任何拥有国际化文本的消息并在继续链接之前将其替换为处理后的文本。第二个拦截器拦截任定义了作者的消息并在使用原始消息恢复链接之前将其打印出来。但因为`I18n`使用`pipelineInner`添加拦截器，`Audit`使用`pipelineOuter`添加拦截器，审计会发生在国际化之前。

因此，如果我们将这两个拦截器同时混入到 Actor，我们将会看到类似如下的示例消息：

```scala
class PrinterActor extends Actor with ReceivePipeline
  with I18nInterceptor with AuditInterceptor {
  override def receive:Receive = {
    case Message(author, text) =>
      println(s"${author.getOrElse("Unknown")} says '$text'")
  }
}

printerActor ! Message(Some("The Dude"), I18nText("EN", "that.rug"))
// The Dude is about to say: I18nText(EN,that.rug)
// The Dude says 'That rug really tied the room together.'

printerActor ! Message(Some("The Dude"), I18nText("EN", "your.opinion"))
// The Dude is about to say: I18nText(EN,your.opinion)
// The Dude says 'Yeah, well, you know, that's just, like, your opinion, man.'
```

## 未处理消息

基于这些行为链接的改变，那些未处理(unhandled)消息会发生什么呢？让我通过一个简单的规则来解释一下。

> **注意：**
>
> 每个未被拦截器处理的消息都会被传递到链路中的下一个拦截器。如果没有任何拦截器处理该消息，则 Actor 当前的行为会处理它，如果该行为也没有处理它，该消息则会向往常一样被未处理方法来处理。

但有些时候希望处理器能够打断链接。你可以通过返回一个`HandledCompletely`消息来显示指示消息已经被拦截器处理完成。

```scala
case class PrivateMessage(userId:Option[Long], msg: Any)

trait PrivateInterceptor {
  this: ReceivePipeline =>
  
  pipelineInner {
    case PrivateMessage(Some(userId), msg) =>
      if(isGranted(userId)) Inner(msg)
      else HandledCompletely
  }
}
```

## 委托之后的处理

但是如果你想在 Actor 处理完消息之后执行一些动作呢(比如度量消息处理的时间)？

为了支持该场景，`Inner`返回类型拥有一个`andAfter`方法，它接收一个代码段用于在随后的内部拦截器处理完成之后执行一些动作。

下面的拦截器实例用于对消息处理计时：

```scala
trait TimerInterceptor extends ActorLogging {
  this: ReceivePipeline =>
  
  def logTimeTaken(time: Long) = log.debug(s"Time taken: $time ns")
  
  pipelineOuter{
    case e => 
      val start = System.nanoTime
      Inner(e).andAfter{
        val end = System.nanoTime
        logTimeTaken(end -start)
      }
  }
}
```

> **注意：**
>
> `andAfter`代码块将会在下一个内部处理器返回之后运行，并且运行在同一线程，因此`andAfter`逻辑闭含拦截器的状态是安全的。

## 与持久化一起使用接收流水线

当与  [*PersistentActor*](http://doc.akka.io/docs/akka/2.4/scala/persistence.html#persistence-scala) 一起使用 `ReceivePipeline` 时需要确保按照以下的顺序混入对应特质：

```scala
class ExampleActor extends PersistentActor with ReceivePipeline{
  /*...*/
}
```

这个顺序是非常重要的，因为这决定了两个特质将如何使用内部的"around(aroundReceive)"方法来实现各自的特性，如果使用其他顺序则不会按预期的方式工作。如果你想学习这具体是如何工作的，可以参考 Scala 的  [type linearization mechanism](http://ktoso.github.io/scala-types-of-types/#type-linearization-vs-the-diamond-problem)。

