---
layout: post
title: RDD가 좋은 이유
date: 2018-10-29 15:10
categories: Spark
---

# RDD가 좋은 이유

누가 Spark가 좋냐고 물어본다면 좋다고 말할 것이다. 그럼 왜 좋은가라고 물어본다면 RDD를 사용하기때문이라고 말할 것이다. 그럼 RDD가 머야?라는 질문에 답변을 하고자 이글을 정리한다.


## RDD이란?

RDD의 정식 명칭은 `Resilient Distributed Datasets`이다. Spark를 이루는 기본연산단위이다. 기존의 Mapreduce을 이용한 방식에서 iteration과 interactive job에 한계를 느껴서 시작한 프로젝트이다.

큰 특징은

- **Immutable**, partitioned collections of records
- 스토리지 -> RDD변환 or RDD -> RDD변환만 가능
- lazy-execution (action에 해당하는 명령어가 실행되야지 transformation이 실행됨)
- Immutalbe하니 Read-Only이다. 그래서 실행계획(lineage)를 그릴 수 있고 최적화 연산을 할 수 있다.
- 실행계획(lineage)가 그려지기때문에 Fault Tolerancer가 능해진다.
- 클러스터 전체에서 공유되는 데이터 형태로 대부분 메모리에 올라가 있음 (=빠르다)

또 알아야될 것은 다음과 같다.

기존 Mapreduce를 사용할때에는 Map, Reduce연산만을 가지고 지지고 볶고 했다. 하지만 RDD를 사용한 Spark는 다양한 요구사항을 처리할 수 있다.(MR, Pregel, DryadLINQ, Iterative MR, SQL, Batched Stream Processiog)

#### 그래서 RDD가 먼가요?라는 질문의 한줄 답변은 다음과 같다.

fault-tolerant & efficient한 램스토리지이다. 

### Reference

- [https://www.slideshare.net/yongho/rdd-paper-review](https://www.slideshare.net/yongho/rdd-paper-review)
- [http://people.csail.mit.edu/matei/papers/2012/nsdi_spark.pdf](http://people.csail.mit.edu/matei/papers/2012/nsdi_spark.pdf)
