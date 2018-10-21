---
layout: post
title: Spark HiveContext FileRead시 주의할 점
date: 2018-10-21 15:10
categories: Spark
---

# Spark HiveContext FileRead시 주의할 점

회사에서는 아직 `Spark1.6`을 사용하고 있어 `hiveContext`를 사용하여 HDFS에 있는 orc, parquet, gz등을 읽을 때가 많다.


이때 `filePath`입력시 주의할 점이 있다.

### HDFS에 basePath아래 file을 전부 읽고 싶은 경우 

file를 읽을때 아래와 같은 코드를 사용한다.

```java
JavaRDD<String> tempRDD = hiveContext.read().format("orc").load(path)
```


```
basePath/1.orc
basepath/2.orc
```

**path자리에 `basePath/*`로 입력하면 file을 읽지 못한다.**

또 이 문제는 Java Spark에서만 발생한다.

#### 해결방법

- `basePath/*.orc`로 확장자 아스타를 넣어주는 방식
- `basePath/`까지만 path로 입력한다.
- fileName에 확장자명이 붙어있지 않으면 된다.
- spark2.x로 업뎃하고 sparkSession을 이용하여 file을 읽는다.

