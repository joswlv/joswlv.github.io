---
layout: post
title: Spark 실행 구성
date: 2018-12-04 15:10
categories: Spark
---

# Spark 실행 구성

### Spark 실행 구성 정리

- 결과 RDD인 counts는 어떤 Action도 수행되지 않은 상태에선 내부적으로 정의된 RDD 객체들의 방향성 비순환 그래프(DAG, Directed Acyclic Graph)를 갖으며, 이것이 나중에 Action을 수행할 때 쓰임
- 각 RDD는 자신이 어떤 타입의 관계를 갖고 있는지에 대한 메타데이터에 따라 하나 이상의 부모 RDD를 가리키는 포인터를 유지, 이런 포인터들은 RDD가 모든 조상들을 추적할 수 있게 해줌
- `toDebugString()` 메소드로 RDD의 가계도 출력 가능
- Spark의 스케줄러는 Action을 수행할 때 필요한 RDD 연산의 Physical Plan을 만듬
- ​Spark의 스케줄러는 연산되는 마지막 RDD에서 시작하여 연산해야 할 것을 역으로 추적, 모든 조상 RDD들을 연산하기 위해 필요한 Physical Plan을 재귀적으로 생성
- 그래프의 각 RDD에 대해 하나의 연산 Stage를 출력하게 되고, 이 Stage는 해당 RDD의 각 Partition들을 위한 Task들을 갖으며, 최종 결과 RDD를 연산해 내기 위해 Stage들은 역순으로 실행
- ​파이프라이닝 외에도 Spark의 내부 스케줄러는 RDD가 이미 Cluster 메모리나 디스크에 Caching되어 있는 경우 RDD 그래프의 가계도를 제거

- 특정 Action을 위해 생성되는 Stage들이 모여 Job을 이룸
즉, count() 같은 Action을 호출할 때마다 하나 이상의 Stage로 구성된 Job이 생성 됨

- 한 번 Stage 그래프가 정의되면 Task들이 만들어지고 사용하는 배포 모드에 따라 다양하게 내부 스케줄러로 전송 됨

- Physical Plan에서 Stage들은 RDD 가계도에 따라 각자 의존성을 가지게 되므로 그에 맞는 순서로 실행


- Stage는 실행되는 Partition은 서로 다르지만 같은 일을 수행하는 Task들을 실행 시킴

- 각 Task는 내부적으로 다음과 같은 동일한 순서에 따라 수행됨

#### 요약하면, Spark의 실행 중에는 다음과 같은 단계들이 발생


1. 사용자 코드가 RDD의 DAG를 정의한다.

> RDD의 연산들은 새로운 RDD를 만들고 이것들은 부모를 참조하게 되면서 이에 따라 그래프가 만들어 진다.

2. DAG가 Action의 Physical Plan으로 변환되게 한다.

> RDD에서 Action을 호출하면 그때는 반드시 연산이 수행되어야 한다. 이는 또한 부모 RDD에게도 연산을 요구하게 된다. Spark의 스케줄러는 RDD들이 필요한 모든 연산을 수행하도록 Job을 제출한다. 이 Job은 하나 이상의 Stage를 갖게 되며, 이는 Task들로 구성된 병렬 집단 연산들을 말한다. 각 Stage는 DAG에서 하나 이상의 RDD들과 연계된다. 하나의 Stage는 파이프라이닝에 의해 여러개의 RDD와 연계될 수도 있다.

3. Task들이 스케줄링되고 Cluster에서 실행된다.

> Stage들은 순서대로 실행되며 RDD의 조각들을 연산하기 위한 Task들을 실행한다. Stage의 최종 단계가 끝나면 Action이 완료된다.