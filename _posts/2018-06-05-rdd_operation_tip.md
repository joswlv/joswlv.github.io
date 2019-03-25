---
layout: post
title: RDD연산할때 있었던 일 (repartition)
date: 2018-06-05 15:10
categories: Spark
---

# RDD연산할때 있었던 일 (repartition)

### 들어가기 전에

회사에서 용량이 30G정도 되는 gz파일 두개의 subtract를 구해 cassandra에 insert하는 작업이 있었다.(꽤 오래전일..)

처음 생각할 수 있는 코드 다음과 같을 것이다.

1. Read 2 gzFile
2. Each file map, filter operation
3. subtract (reduce)
4. save to cassandra

클러스터의 memory를 많이 사용할 수 있고 spark버전이 2.0이상이면 문제 없이(?) 많은 리소스를 사용하면서 잘 동작하겠지만 나에게 주어진 환경은 spark1.6버전 (`spark 2.0이하 버전에서는 container 메모리가 일정 수준을 넘어가면 yarn에서 강제로 죽여버린다.` [참고](https://gsamaras.wordpress.com/code/memoryoverhead-issue-in-spark/)) 한정된 리소스였다. 

그래서 위 코드 plan으로 코드를 작성하면 subtract연산을 할때 많은 데이터가 메모리 위에 존재하게 되고 다음과 같은 에러를 보게 된다. 

![]({{ site.url }}/images/spark_yarn_mem_overhad.png)

다시 생각해보면 MapReduce작업이고 Reduce작업 할때 병목현상이 생겨 메모리를 많이 사용하게 된다. 

이러한 병목현상을 줄이기 위해서 `reduceByKey`, `groupByKey`, `MapPartitions`, `RepartitionAndSortWithinPartitions`이런 연산들이 있다.

> 여기서 `groupByKey`연산은 조심해서 사용해야하는데, groupBy연산은 shuffle이 발생한 뒤에 single in-memory에서 연산을 하므로 memory를 많이 사용하게 된다.

그래서 결론은 `repartitionAndSortWithinPartitions`연산을 추가해서 문제를 해결하였다.

> Spark-Sql를 사용하면 이런 튜닝은 자동으로 해준다. 똑같은 연산을 Dataframe join으로 풀어보면 실행계획 중에 `repartitionAndSortWithinPartitions`이 들어간 것을 확인 할 수 있었다.

### Repartitions 이야기

#### 1. `MapPartitoins`

map연산과 `HashPartitoiner`를 이용해서 hash함수에 key를 통과 시켜 같은 HashCode를 가지는 데이터는 같은 partition에 넣는 작업을 한다. 

이게 `groupByKey`연산과 가장 크게 다른점은 record를 하나의 memory에 모으지 않고 iterator를 사용해 stream처리해 memory를 적게 사용한다.

좋은점은 iterator를 직접 만들어서 partitions되는동안에 logic을 넣을 수 있다. 이렇게 되서 얻는 장점은 logic에 사용되는 memory 사용량을 줄일 수 있다.

```scala
val outputRDD = partitionedRDD.mapPartitions(v => new CustomIterator(v))

  class CustomIterator(iter: Iterator[(String, String)]) extends Iterator[(String, String)] {

    var lastKeyBase = null: String
    var attr = null: String
  
    def hasNext : Boolean = {
      iter.hasNext
    }
    
    def next : (String, String) = {
      var record = iter.next
      var key = record._1.split("\\|")
      var keyBase = key(0)
      var keyFlag = key(1)
        
      // Parse value
      var value = record._2.split(",")
      var inAttr = value(3)
        
      if (lastKeyBase != null && !lastKeyBase.equals(keyBase)) {
        // Reset group attribute
        attr = null
        lastKeyBase = keyBase
      }
        
      if (keyFlag.equals("A") && attr == null) {
        attr = inAttr
      }
        
      (keyBase, value(0) + "," + value(1) + "," + value(2) + "," + (if (attr != null) attr else inAttr))
    }
  }
```

#### 2. `repartitionAndSortWithinPartitions `

`repartitionAndSortWithinPartitions` 연산은 글자 그대로 partition연산을 할때 key에대해서 sorting을 해준다. 

이 연산이 가지는 장점은 여러개의 RDD에대해서 join연산을 할때 장점을 가진다. 
subtarct연산을 할때도 역시 장점을 가진다 key에 대해 sorting이 되어 있으니, 동일 key가 발견되면 해당 key 뒤 내용을 전혀 볼 필요가 없으니 말이다.


### 3. KeyBasePartitioner

hashcode를 통한 key로 partition을 할때 사용하는 partitoner

```scala
class KeyBasePartitioner(partitions: Int) extends Partitioner {

	override def numPartitions: Int = partitions

	override def getPartition(key: Any): Int = {
	//KEY를 자유롭게 설정하면 된다.
		Math.abs(key.hashCode() % numPartitions)
	}
}
```

### Refrence

- [http://technology.finra.org/code/using-spark-transformations-for-mpreduce-jobs.html](http://technology.finra.org/code/using-spark-transformations-for-mpreduce-jobs.html)
