#### 添加国内的maven库
在”~/.sbt”下创建repositories文件
```
[repositories]
local　　　　　　　　　　　　　　　　　　　　　　　　　# 本地ivy库
maven-local: file://~/.m2/repository　　　　　　　# 本地Maven库　
osc: http://maven.oschina.net/content/groups/public/    #开源中国的maven库，用于加快速度
typesafe: http://repo.typesafe.com/typesafe/ivy-releases/, [organization]/[module]/(scala_[scalaVersion]/)(sbt_[sbtVersion]/)[revision]/[type]s/[artifact](-[classifier]).[ext], bootOnly
sonatype-oss-releases
maven-central
sonatype-oss-snapshots
```
#### sbt项目目录
```
├── src
│　 ├── main
│　 │　 ├── java
│　 │　 ├── resources
│　 │　 └── scala
│　 ├── test
│　 │　 ├── java
│　 │　 ├── resources
│　 │　 └── scala
├── build.sbt
├── project
│　 ├── build.properties
│　 ├── plugins.sbt
```
#### build.sbt
```
name := "hello"      // 项目名称

organization := "xxx.xxx.xxx"  // 组织名称

version := "0.0.1"  // 版本号

scalaVersion := "2.10.6"   // 使用的Scala版本号

// 添加项目依赖
libraryDependencies += "ch.qos.logback" % "logback-core" % "1.0.0"

libraryDependencies += "ch.qos.logback" % "logback-classic" % "1.0.0"

// 或者

libraryDependencies ++= Seq(
                            "ch.qos.logback" % "logback-core" % "1.0.0",
                            "ch.qos.logback" % "logback-classic" % "1.0.0",
                            ...
                            )

// 添加测试代码编译或者运行期间使用的依赖
libraryDependencies ++= Seq("org.scalatest" %% "scalatest" % "1.8" % "test")
```
