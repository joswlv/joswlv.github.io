---
layout: post
title: Spark-sumbit --files option
date: 2019-04-24 13:10
categories: Spark
---

# Spark-sumbit --files option주의사항

Spark Application을 `cluster`모드로 제작할 때 SparkConf 정보 등과 같이 다양한 리소스 파일을 읽는 경우가 있다. 

이때 사용하면 좋은 옵션이 `--files` 옵션이다.

(리소스파일 hadoop에 올리는 방법도 있지만, 매우 낭비가 큰 방식이라 선호하지는 않는다.)

deploy-mode를 `client`모드를 사용할때는 dirver에서 로컬 파일을 읽는 방식으로 리소스 파일을 읽어 사용하면 되지만 `cluster`모드에서는 어떤 노드가 dirver노드가 될지 모르기 때문에 로컬 파일를 읽는 방식 처럼 사용하기는 힘들다.

이때 사용하는 방식이 spark-submit옵션 중에 `--files` 옵션을 사용하는 것이다.

아래와 같이 `--files` 다음에 리소스파일 path를 주면 사용이 가능하다.

```bash
    spark-submit \
    		--class ${class}  \
    		--name "${appName}_${date}_${type}" \
    		--master yarn \
    		--deploy-mode cluster \
    		--files ${filteringFile} \
    		${runjar} 
```	

[spark-core소스](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/deploy/SparkSubmitArguments.scala#L547)를 보면 `--files`로 deploy된 리소스파일은 각각의 executors에서 사용이 가능하다고 나온다. 즉 spark context로 배포되기때문에 어느 시점에서 로컬 파일처럼 읽어도 사용이 가능하다.

spark application에서  `Files.newBufferedReader(Paths.get(fileName))` 으로 읽어서 사용하면 된다.


### 다만 주의할 점은 다음과 같다.

```scala
//files로 배포되고 각 executor에 저장된 localFilePath
val executorLocalFilePath = SparkFiles.get(fileName);
    
//아래 코드는 NotFoundFileException을 발생시킨다.
Source.fromFile(executorLocalFilePath)
```    

`--files`옵션으로 배포된 file을 읽을때는 Path에 fileName만 주면 상대경로로 file을 찾아준다. 

절대경로 file 찾기란 쉽지 않으니,, 상대경로를 사용하자!!

```scala
//files로 배포되고 소스에서 사용할려면 fileName만 주면 된다.
Source.fromFile(fileName)
```

(이것 때문에 많은 시간을 보냈다.ㅠㅠ)


### 추가 사항 FileName `alias`설정

```bash
    spark-submit \
    		--class ${class}  \
    		--name "${appName}_${date}_${type}" \
    		--master yarn \
    		--deploy-mode cluster \
    		--files fileName#aliasName \
    		${runjar} 
```	

`--files`옵션 파라미터에 `#`을 사용하면 `#`뒤 String이 alias로 사용된다.

`alias`는 `cluster모드`에서만 사용이 가능하고 소스에서 사용할 때는 `aliasName`으로 호출해서 사용한다. (fileName으로는 접근 안됨)

```scala
//files로 배포되고 소스에서 사용할려면 aliasName을 넣어줘야한다.
Source.fromFile(aliasName)
```