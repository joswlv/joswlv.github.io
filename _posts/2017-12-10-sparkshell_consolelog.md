---
layout: post
title: Spark Shell log제거 (ver1.5)
date: 2017-12-10 15:10
categories: Spark
---

# SparkSehll 로그 제거

### 1. conf/log4j.properties 수정

conf/log4j.properties에서 log level를 수정한다.

```
log4j.rootCategory=INFO, console

to

log4j.rootCategory=ERROR, console
```

### 2. Spark-shell에서 off하기

```
import org.apache.log4j.Logger
import org.apache.log4j.Level

Logger.getLogger("org").setLevel(Level.OFF)
Logger.getLogger("akka").setLevel(Level.OFF)
```

