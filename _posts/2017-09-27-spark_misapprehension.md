---
layout: post
title: Spark에 대한 오해
date: 2017-09-27 15:10
categories: Spark
---

# Spark에 대한 오해

#### 대표적인 오해

- Spark는 in-memory technology이다?
- Spark는 hadoop MR보다 100배 빠르다?
- Spark는 이분야에서 데이터 프로세싱을 완전히 새로운 접근을 소개했다?


### 1. Spark는 in-memory technology가 아니다.

in-memory technology라고 하는 것은, 데이터를 처리하는 동안 RAM에 persist 하는것 이라고 한다. 

하지만 Spark의 겨우 모든 겨우 데이터를 persist하는 것이 아니라 cache를 한다.

- 요즘 OS단에서 데이터를 메모리에 로딩하지 않고 바로 disk에서 바로 프로세싱이 가능하게 하는 API는 없다!! 그러므로 데이터 프로세싱을 한다는 것은 굳이 언급하지 않아도 in-memory processing이다.
- Spark는 LRU eviction rules를 사용한다. 데이터가 메모리에 충분히 들어갈 경우에는 in-memory technology가 맞지만 그렇지 않을 경우에는 LRU룰을 따른다. 요즘 대부분의 RDBMS 또한 LRU룰을 이용하지만 in-memory technology라고 부르지 않는다. 또한 Linux IO 역시 모든 IO operations이 OS IO cashe(이 역시 LRU캐쉬)를 통과하고 있다.
- Spark는 모든 transforamtion연산을 in-memory에서 하지 않는다. shuffle시에 데이터를 disk에 쓰도록 설계되어 있다. 가령 SparkSQL query에서 group by를 하거나, RDD를 PairRDD로 변경하거나, aggregationbyKey를 하면, Spark에게 key의 hash된 값을 기반으로 partition들 사이에서 데이터를 distribute하게 된다. shuffle은 2가지 phase로 이루어지는데, 1) map, 2) reduce이다. map은 단지 key의 hash값을 계산하고, N개의 파일로 local filesystem상에 output을 만든다. reduce는 map의 데이터를 가져와 merge를 해서 새로운 partition을 만든다. 만약 M개의 partitions을 가지고, N개의 pair RDD쌍으로 transform을 한다면, M*N개의 파일을 local filesystem에 생성한다. 파일을 줄이기 위한 최적화가 있긴하다. 하지만 그런 앞단의 작업이 있음에도 불구하고, 결국에 HDD에 데이터를 놓기 위해서는 shuffle이 필요하다는 사실만은 변함이 없다.

**따라서** Spark는 in-memory technology가 아니다.

- spark는 효과적으로 in-memory LRU cache를 할 수 잇게 해주는 기술이다.
- Spark는 built-in persistence functionality를 가지지 않았다.
- Spark는 shuffle과정동안에 모든 dataset을 local filesystem에 놓는다.

### 2. Hadoop MR보다 100배 빠르다?

![]({{ site.url }}/images/SparkBenchmark.png)

Spark 메인페이지에서 볼 수 있는 위와 같은 성능 측정 결과는 Logistic regreesion에 대한 것이다. 머신러닝에 대한 일을 처리할 경우 테스트하여 성능 결과를 측정해 놓은 것이다.

보통 머신러닝에서는 동일한 dataset에 대해서 repeatedly iteration을 많이 하곤한다. 이 경우 Spark의 LRU 캐쉬를 이용한 in-memory cache가 빛이나는 경우이다.!!

반복적으로 같은 dataset에 여러번 접근할 경우에 처음에만 읽고 원할때 마다 데이터에 접근 할 수 있는데 결국 memory에 있는 것을 읽기만 하면 된다. 이 경우는 BestCase이다.

그러나 불행하게도, Hadoop상에서 위와 같은 반복작업이 실행될 경우에는 HDFS cache 능력을 잘 활용하지 못하는거 같다. ([Haddop-cache](http://hadoop.apache.org/docs/r2.4.1/hadoop-project-dist/hadoop-hdfs/CentralizedCacheManagement.html))


대략적으로 3x ~ 4x배 정도 Spark가 빠르다고 여겨지는데,

- Spark는 startup time이 더 빠르다. (Spark는 thread를 fork하고, MR은 JVM을 새로 띄운다.)
- Spark는 shuffle이 더 빠르다. Spark는 shuffles동안에 한번만 HDD에 데이터를 쓰는데, MR은 이것을 2배 한다.
- Spark는 workflows가 더 빠르다. MR의 workflow는 MR Jobs의 하나의 시리즈이고, 각각 단계에서의 data를 HDFS에 저장을 한다. 반면에 Spark는 DAGs와 pipelining을 지원하는데, 중간 데이터 없이 복잡한 workfows를 허용하게 해준다. (물론 사용자가 shuffle을 필요로 하지 않을 경우에..)
- Caching이 더 낫다. 이것은 약간 논쟁이 되고 있는데, 지금은 HDFS또한 cache을 할 수 있기 때문이다. 그러나 대체로 Spark의 cache가 매우 좋다. 특히 SparkSQL에 더욱 그렇다. Spark SQL은 최적화되어지니 column-oriented 형태로 데이터를 캐쉬한다.

결론적으로 짧은 실행의 job의 경우에는 100x배까지 빠를수는 있어도 보통의 실제 경우의 작업에서는 Hadoop의 성능보다 최대 2.5 ~3배를 넘지 않을 것이다.


### 3. Spark는 이분야에서 데이터 프로세싱을 완전히 새로운 접근을 소개했다?

Spark에 의해서 혁명적으로 새로운것이 소개된게 없다. 효과적으로 LRU cache를 이용하고 data processing pipelining을 이용했지만 비단 Spark만 그랫던것은 아니다.

만약에 이 문제에 대해서 마음을 좀 열어둔다면, 너는 MPP databases에 의해서 소개되어진( query execution pipelinig, no itermediate data materializaion, LRU cache) 과 거의 같은 컨셉이라는것을 알아차릴 수 있다.

물론 Spark는 이것을 open source로 해서 무료로 사용할 수 있게 제공하고 있다. ( 대부분의 회사들이 MPP database가 좋다고 해도, 엔터프라이즈 MPP 을 돈 주고 쓰고 있지 않는다.. )



#### 좋은말

In the end, I would like to recommend you not to trust everything you hear from the media. Trust the subject matter experts, they are usually the best persons to ask.


### Reference

- [https://0x0fff.com/spark-misconceptions/](https://0x0fff.com/spark-misconceptions/)
