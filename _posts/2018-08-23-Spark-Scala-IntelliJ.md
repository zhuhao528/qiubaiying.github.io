---
layout:     post
title:      Spark-Scala-IntelliJ开发环境调试
subtitle:	  Spark IntelliJ 调试
date:       2018-08-14
author:     Neil
header-img: img/post-bg-nomal.jpg
catalog: 	 true
tags:
    - Spark
    - 大数据
---

# 包依赖配置文件
 
 build.sbt 关于scala version和spark包引入的依赖版本

```
name := "SparkApi_Test"

version := "1.0"

scalaVersion := "2.11.7"

libraryDependencies += "org.apache.spark" %% "spark-core" % "2.1.0"
```

# 测试代码

```
import scala.math.random
import org.apache.spark._

object SparkPi {
    def main(args: Array[String]) {
      val conf = new SparkConf().setAppName("Spark Pi").setMaster("local")
      val spark = new SparkContext(conf)
      val slices = if (args.length > 0) args(0).toInt else 2
      val n = math.min(100000L * slices, Int.MaxValue).toInt // avoid overflow
      val count = spark.parallelize(1 until n, slices).map { i =>
        val x = random * 2 - 1
        val y = random * 2 - 1
        if (x * x + y * y < 1) 1 else 0
      }.reduce(_ + _)
      println("Pi is roughly " + 4.0 * count / n)
      spark.stop()
    }
  }
```
# 执行结果

![刚进大门](https://ws1.sinaimg.cn/large/006tNbRwly1fujp0ador4j30yn087acr.jpg)





