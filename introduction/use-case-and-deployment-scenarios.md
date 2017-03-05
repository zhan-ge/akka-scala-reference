# 案例与部署方案

## 如何使用和部署 Akka

Akka 可以以不同的方式使用：

- 作为一个库：向常规的 JAR 包一样用在 Classpath 或放在 WEB-INF/lib 中供 web 应用使用，
- 使用 [sbt-native-packager](https://github.com/sbt/sbt-native-packager) 打包。
- 使用 [Lightbend ConductR](http://www.lightbend.com/products/conductr) 打包或部署。

## Native Packager

[sbt-native-packager](https://github.com/sbt/sbt-native-packager) 是一个用于创建各类应用分发包的工具，包括 Akka 应用。

在`project/build.properties`定义 SBT 版本：

```scala
sbt.version=0.13.7
```

在`project/plugins.sbt`中添加 [sbt-native-packager](https://github.com/sbt/sbt-native-packager) 插件：

```scala
addSbtPlugin("com.typesafe.sbt" % "sbt-native-packager" % "1.0.0-RC1")
```

在`build.sbt`文件中使用包属性配置，同时可以根据需要指定`mainClass`:

```scala
import NativePackagerHelper._
 
name := "akka-sample-main-scala"
 
version := "2.4.17"
 
scalaVersion := "2.11.8"
 
libraryDependencies ++= Seq(
  "com.typesafe.akka" %% "akka-actor" % "2.4.17"
)
 
enablePlugins(JavaServerAppPackaging)
 
mainClass in Compile := Some("sample.hello.Main")
 
mappings in Universal ++= {
  // optional example illustrating how to copy additional directory
  directory("scripts") ++
  // copy configuration files to config directory
  contentOf("src/main/resources").toMap.mapValues("config/" + _)
}
 
// add 'config' directory first in the classpath of the start script,
// an alternative is to set the config file locations via CLI parameters
// when starting the application
scriptClasspath := Seq("../config/") ++ scriptClasspath.value
 
licenses := Seq(("CC0", url("http://creativecommons.org/publicdomain/zero/1.0")))
```

> **注意：**
>
> 使用`JavaServerAppPackaging`，而不是`AkkaAppPackaging`(之前称为`packageArchetype.akka_application`)，因为后者没有前者轻量及质量保证。

使用 SBT 任务来打包应用。

然后可以启动应用(类 Unix 系统中)：

```bash
cd target/universal/
unzip akka-sample-main-scala-2.4.17.zip
chmod u+x akka-sample-main-scala-2.4.17/bin/akka-sample-main-scala
akka-sample-main-scala-2.4.17/bin/akka-sample-main-scala sample.hello.Main
```

使用`Ctrl-C`来中断并退出应用。

在 Windows 系统中可以使用`bin\akka-sample-main-scala.bat`脚本来启动应用。

## 使用 Docker 容器

同时可以再 Docker 中使用 Akka 远程 Actor 和 Akka 集群。但在使用 Docker 时需要额外注意网络配置，详细细节可以参考这里：[*Akka behind NAT or in a Docker container*](http://doc.akka.io/docs/akka/2.4/scala/remoting.html#remote-configuration-nat)。

使用 Akka 集群和 Docker 创建项目可以参考 [activator template](https://www.lightbend.com/activator/template/akka-docker-cluster) 中的 ["akka-docker-cluster" ](https://www.lightbend.com/activator/template/akka-docker-cluster) 实例。

