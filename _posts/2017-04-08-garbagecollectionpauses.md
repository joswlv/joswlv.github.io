---
layout: post
title: [Cassandra] Garbage collection pauses
date: 2017-04-08 09:04
categories: Cassandra
---

# 장애현상

**Using cassandra version : 2.1.8**

```
WARN  [GossipTasks:1] 2017-03-21 17:11:05,462 FailureDetector.java:249 - Not marking nodes down due to local pause of 9594745097 > 5000000000
INFO  [GossipTasks:1] 2017-03-21 17:11:05,462 Gossiper.java:968 - InetAddress /10.161.26.248 is now DOWN
INFO  [Service Thread] 2017-03-21 17:11:05,463 GCInspector.java:252 - ConcurrentMarkSweep GC in 9487ms.  CMS Old Gen: 3039553840 -> 3027311712; Par Eden Space: 833093632 -> 0; Par Survivor Space: 46277016 -> 0
```

이런 로그를 찍고 서버가 `down` 되었다 살았다를 반복하였다.

# 원인

* [https://docs.datastax.com/en/landing_page/doc/landing_page/troubleshooting/cassandra/gcPauses.html](https://docs.datastax.com/en/landing_page/doc/landing_page/troubleshooting/cassandra/gcPauses.html)

`Garbage collection pause`라고 한다. 이 현상이 있으면 해당 노드가 다른 노드에 down 된 것처럼 보일 수 있다고 한다.

원인은 아래 4가지 정도가 있다고 한다.

* If the problem is recent, check for any recent applications changes.
* Excessive tombstone activity: often caused by heavy delete workloads.
* Large row updates or large batch updates: reduce the size of the individual write below 1 Mb (at the most).
* Extremely wide rows: manifests as problems in repairs, selects, caching, and elsewhere.

지금 운영 중인 카산드라 특성상 비슷한 TTL이 걸려 있는 데이터들이 많은데, 한달전에 인입된 데이터가 `heavy delete`되면서 발생한 문제인것으로 추측한다.

`Not marking nodes down due to local pause` 로그는 pause 현상이 너무 오래 지속되었을 때 node down이 계속 된 것처럼 보이기 때문에 client에서 심각한 에러가 발생했다고 볼 수 있어서 최대 시간(5초) 보다 크면 node down 표시를 skip 하겠다 라고 알려주는 로그라고한다.

`Garbage collection pause`현상은 `2.1.6`버전 부터 패치되어 최대시간(5초)동안 서버가 down되었다 살았다를 반복한다.

[https://issues.apache.org/jira/browse/CASSANDRA-9183](https://issues.apache.org/jira/browse/CASSANDRA-9183)


# 해결방안

해결 방안은 크게 두가지 방법이 존재한다.

1. Java 1.7부터 도입된 G1 방식으로 튜닝
2. Cassandra 메모리 설정을 바꾸는 방법

1번은 현재 사용하고 있는 CMS GC방식이 아닌 Java 1.7부터 도입된 G1 방식으로 튜닝한다.

**[https://medium.com/@mlowicki/move-cassandra-2-1-to-g1-garbage-collector-b9fb27365509](https://medium.com/@mlowicki/move-cassandra-2-1-to-g1-garbage-collector-b9fb27365509)**


datastax 에서는 2번 방법 위주로 튜닝 가이드를 제공하고 있다.
`MAX_HEAP_SIZE`, `HEAP_NEWSIZE`를 조정하는 방법이다. 

**[https://docs.datastax.com/en/cassandra/2.1/cassandra/operations/ops_tune_jvm_c.html](https://docs.datastax.com/en/cassandra/2.1/cassandra/operations/ops_tune_jvm_c.html)**
 
`cassandra-env.sh` 설정을 보면 이런 계산식에 의해 default 세팅이 되어 있다.

* `MAX_HEAP_SIZE` = max(min(1/2 ram, 1024MB), min(1/4 ram, 8GB))

* `HEAP_NEWSIZE` = min(max_sensible_per_modern_cpu_core * num_cores, 1/4 * heap size)    
(max_sensible_per_modern_cpu_core 는 100으로 세팅됨) 
 
카산드라 서버들이 모두 24 CPU core, 16G 메모리를 가지고 있으므로
`MAX_HEAP_SIZE`는 4G, `HEAP_NEWSIZE`는 1G 로 세팅되어 있다.
 
datastax 가이드 추천대로라면
`MAX_HEAP_SIZE`= 8G,  `HEAP_NEWSIZE`=2G 정도까지 늘릴 수 있을 것 같다.
그리고 현재 서버 메모리를 40%정도만 사용하고 있으므로 큰 영향은 없을 것 같다.



#### Java GC관련 문서 

* [http://d2.naver.com/helloworld/1329](http://d2.naver.com/helloworld/1329)