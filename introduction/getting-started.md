# 准备开始

## 必要准备

Akka 需要你的机器上已安装 [Java 8](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 或更新版本。

[Lightbend Inc.](http://www.lightbend.com/) 为 Akka 和 [Lightbend Reactive Platform](http://www.lightbend.com/platform) 中的相关项目比如 Scala、Play! 提供了一个商业版本，以免你的项目还不能及时升级到 Java 8。同时还包含一些商业版特性和扩展库。

## 获取入门指南和工程模板

学习 Akka 最好的方式就是下载  [Lightbend Activator](http://www.lightbend.com/platform/getstarted) 并尝试其中的项目模板。

## 下载

有多种方式来下载 Akka。你可以将其作为 Lightbend Platform 的一部分下载。也可以下载包含所有模块的全部分发包。或者使用类似 Maven 或 SBT 这样的构建工具从 Akka 的 Maven 仓库下载需要的依赖。

## 模块

Akka 是非常模块化的，有多个包含不同特性的 JAR 包组成：

- akka-actor：经典 Actor，Typed Actor，IO Actor 等。
- akka-agent：Agent，与 Scala STM 集成。
- akka-camel：Apache Camel 集成。
- akka-cluster：集群成员管理，弹性路由。
- akka-osgi：在 OSGI 容器中使用 Akka 的工具包。
- akka-osgi-aries：Aries 配置 Actor 系统的蓝图。
- akka-remote：远程 Actor。
- akks-slf4j：SLF4J 日志库(事件总线监听器)。
- akka-stream：响应式流处理。
- akka-testkit：Actor 系统测试工具集。

除了这些稳定的模块，还有其他一些模块正以他们各自的方式进入稳定阶段，但目前仍会被标记为“实验性”。这并不意味着他们符合预期的功能，主要是为了表示他们的 API 还没有固定下来直至冻结状态。你可以在邮件列表中反馈这些模块的使用情况以加速该进程。

- akka-contrib：一个提交的混合分类，未确定是否要移入核心模块，可以查看 [*External Contributions*](http://doc.akka.io/docs/akka/2.4/contrib/index.html#akka-contrib) 了解更多细节。

JAR 包的名字可能类似像`akka-actor_2.11-2.4.17.jar`这样，其他模块也类似。

可以在 [*依赖*](http://doc.akka.io/docs/akka/2.4/dev/building-akka.html#dependencies) 一节查看各个模块间的相互依赖关系。

## 使用已释出的分发包

从 [http://akka.io/downloads](http://akka.io/downloads) 下载需要的释出版本并解压。

## 使用快照版本

Akka 的夜间快照版本会被发布到  [http://repo.akka.io/snapshots/](http://repo.akka.io/snapshots/) 并使用 SNAPSHOT 或时间戳来标记版本号。你可以选择一个已被时间戳标记的版本来使用或决定何时升级到最新版本。

> **警告：**
>
> 慎重选择你需要的版本，比如 SNAPSHOT、夜间版、里程碑释出版。

## 使用构建工具

Akka 可以使用任何支持 Maven 仓库的构建工具。

## Maven 仓库

从 Akka 2.1-M2 及之后的版本：[Maven Central](https://repo1.maven.org/maven2/)。

之前的版本：[Akka Repo](http://repo.akka.io/releases/)。

## 通过 Maven 使用 Akka

通过 Maven 使用 Akka 最简单的方式是找到 [Lightbend Activator](http://www.lightbend.com/platform/getstarted) 中一个名为 [Akka Main in Java](http://www.lightbend.com/activator/template/akka-sample-main-java) 的教程。

由于 Akka 已经发布到了 Maven 中心仓库(从 2.1-M2 开始)，可以直接将依赖添加到 POM 文件中。比如，添加`akka-actor`的依赖：

```xml
<dependency>
  <groupId>com.typesafe.akka</groupId>
  <artifactId>akka-actor_2.11</artifactId>
  <version>2.4.17</version>
</dependency>
```

对于快照版本，可以以相同的方式添加快照仓库：

```xml
<repositories>
  <repository>
    <id>akka-snapshots</id>
      <snapshots>
        <enabled>true</enabled>
      </snapshots>
    <url>http://repo.akka.io/snapshots/</url>
  </repository>
</repositories>
```

**注意**：快照版本会以 SNAPSHOT 和时间戳标记两种方式发布。

## 通过 SBT 使用 Akka

通过 SBT 使用 Akka 最简单的方式是找到 [Lightbend Activator](http://www.lightbend.com/platform/getstarted) 中的 SBT [模板](https://www.lightbend.com/activator/templates)。

通过 SBT 使用 Akka 的要点总结：

SBT 安装说明：[http://www.scala-sbt.org/release/tutorial/Setup.html](http://www.scala-sbt.org/release/tutorial/Setup.html)。

`build.sbt`文件：

```scala
name := "My Project"
 
version := "1.0"
 
scalaVersion := "2.11.8"
 
libraryDependencies += "com.typesafe.akka" %% "akka-actor" % "2.4.17"
```

**注意**：上面的`libraryDependencies`配置支持 SBT `v0.12.x`或更高版本。如果你使用老版本的 SBT，`libraryDependencies`部分看起来会是这样：

```scala
libraryDependencies +=
  "com.typesafe.akka" % "akka-actor_2.11" % "2.4.17"
```

对于快照版本，可以以相同的方式添加快照仓库：

```scala
resolvers += "Akka Snapshot Repository" at "http://repo.akka.io/snapshots/"
```

## 通过 Gradle 使用 Akka

最少需要  [Gradle](https://gradle.org/) 1.4 版本并使用 [Scala plugin](http://www.gradle.org/docs/current/userguide/scala_plugin.html)：

```gradle
apply plugin: 'scala'
 
repositories {
  mavenCentral()
}
 
dependencies {
  compile 'org.scala-lang:scala-library:2.11.8'
}
 
tasks.withType(ScalaCompile) {
  scalaCompileOptions.useAnt = false
}
 
dependencies {
  compile group: 'com.typesafe.akka', name: 'akka-actor_2.11', version: '2.4.17'
  compile group: 'org.scala-lang', name: 'scala-library', version: '2.11.8'
}
```

对于快照版本，可以以相同的方式添加快照仓库：

```gradle
repositories {
  mavenCentral()
  maven {
    url "http://repo.akka.io/snapshots/"
  }
}
```

## 通过 Eclipse 使用 Akka

创建 SBT 项目并使用 [sbteclipse](https://github.com/typesafehub/sbteclipse) 生成 Ecpilse 项目。

## 通过 Intellij IDEA 使用 Akka

创建 SBT 项目并使用 [sbt-idea](https://github.com/mpeltonen/sbt-idea) 生成 Intellij IDEA 项目。

## 通过 NetBeans 使用 Akka

创建 SBT 项目并使用 [nbsbt](https://github.com/dcaoyuan/nbsbt) 生成 NetBeans 项目。

同时需要使用 [nbscala](https://github.com/dcaoyuan/nbscala) 在 IDE 中提供 Scala 的通用支持。

## 不要使用 Scala 的 -optimize 编译器标记

> **警告：**
>
> Akka 并未通过 Scala 编译器标记 -optimize 来编译或测试。尝试过的用户已经报告了奇怪的行为。

## 通过源码编译

Akka 使用 Git 并托管在 [Github](https://github.com/) 上。

- Akka：从 [https://github.com/akka/akka](https://github.com/akka/akka) 复制 Akka 仓库。

继续阅读 [*Building Akka*](http://doc.akka.io/docs/akka/2.4/dev/building-akka.html#building-akka) 了解更多细节。

## 需要帮助？

如果你有任何疑问可以在 [Akka 邮件列表](https://groups.google.com/group/akka-user) 获得帮助。

或者寻求 [商业版支持](https://www.lightbend.com/)。

感谢称为 Akka 社区的一员。