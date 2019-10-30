# Build Settins

制定时间： 2019 年 6 月 29 日
目的：统一一下版本号和标准选项。

## `build.sbt`

```scala
libraryDependencies ++= {
  val akkaVersion = "2.5.23"
  val httpVersion = "10.1.8"
  Seq(
    "com.typesafe.akka" %% "akka-stream"      % akkaVersion,
    "com.typesafe.akka" %% "akka-http"  % httpVersion,
    "com.typesafe.akka" %% "akka-http-spray-json"  % httpVersion,
    "com.typesafe.akka" %% "akka-slf4j" % akkaVersion,
    "ch.qos.logback"    %  "logback-classic" % "1.2.3",
    "com.typesafe.akka" %% "akka-testkit" % akkaVersion % Test,
    "org.scalatest"     %% "scalatest" % "3.0.8" % Test,
    "com.typesafe.slick" %% "slick" % "3.3.2"
  )
}
```

## `scala.sbt`

```scala
scalaVersion := "2.13.0"

scalacOptions ++= Seq(
  "-encoding", "utf8", // Option and arguments on same line
  "-Xfatal-warnings",  // New lines for each options
  "-deprecation",
  "-unchecked",
  "-language:implicitConversions",
  "-language:higherKinds",
  "-language:existentials",
  "-language:postfixOps"
)
```

## `project/build.properties`

```text
sbt.version=1.2.8
```

## `project/plugins.sbt`

This is an optional and to be tested.

```scala
resolvers += Classpaths.typesafeReleases

addSbtPlugin("com.eed3si9n" % "sbt-assembly" % "0.14.9")

addSbtPlugin("com.typesafe.sbt" % "sbt-native-packager" % "1.3.22")
```
