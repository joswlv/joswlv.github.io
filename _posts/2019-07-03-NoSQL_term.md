---
layout: post
title: NoSQL 아키텍처를 이해기 위한 용어정리
date: 2019-07-03 20:10
categories: NoSQL
---

# NoSQL 아키텍처를 이해기 위한 용어정리

### 개요

NoSQL, 분산스토리지를 사용하면서 많이 들었던 용어를 정리할 필요가 있다고 느껴 정리하고자 한다.

## 1. Bloom Filter

블룸필터(Bloom Filter)는 1970년도에 Burton H. Bloom이 고안한 것으로 공간 효율적인 probabilistic data structure이며 구성요소가 집합의 구성원인지 점검하는데 사용된다. False positive(없다고 예측하고 실제로 없는경우)들이 가능하며, false negative(없다고 예측하고 실제로 있는경우)들은 불가능하다. 요소들은 집합에 추가될 수 있으나 제거는 되지 않는다. 그리고 블룸 필터는 메모리를 좀 더 사용함으로써 자주 호출되는 비싼 함수들의 성능을 크게 향상시키는 방법 중에 하나이다.

(즉 집합에 원소가 없으면 진짜 없다는 것을 보장해준다.)

### **구성 요소들과 작동 방식**

#### 구성 요소

- 해시(hash) 함수 : 해시 알고리즘 구현체.
- Bloom Filter Key : 해시 알고리즘에 의해 반환된 KEY.
- Bloom Filter Index : Bloom Filter Key를 임의의 크기의 비트들의 블록으로 나눈 하나 하나.
- 블룸 필터는 두가지를 제공 : 하나는 추가하는 add(), 하나는 존재하는지에 대한 isExist().
- fFalse positive(없다고 예측하고 실제로 없는경우), false negative(없다고 예측하고 실제로 있는경우)
- 작동 방식 예를 들어, m(bloom filter의 bit 사이즈) = 10, k(hashing functions의 수) = 2로하자. 2개의 해시 함수를 h1, h2 하고 Bloom Filter 배열은 아래와 같다.

```
    +-+-+-+-+-+-+-+-+-+-+
    |0|0|0|0|0|0|0|0|0|0|
    +-+-+-+-+-+-+-+-+-+-+
     0 1 2 3 4 5 6 7 8 9
``` 

데이터 A를 삽입한다고 가정하면, 데이터 A를 두 해시 함수 h1, h2를 거쳐 해시 값을 계산한다. 만일 3=h1(A), 7=h2(A) 였다고 하자. 이 해시 값 3, 7에 해당하는 곳에 1을 셋팅한다. 이제 배열은 아래와 같아진다.

```
    +-+-+-+-+-+-+-+-+-+-+
    |0|0|0|1|0|0|0|1|0|0|
    +-+-+-+-+-+-+-+-+-+-+
     0 1 2 3 4 5 6 7 8 9
```

그 다음 데이터 A를 검색한다면, 해시 함수를 거치면 3=h1(A), 7=h2(A)이다. 3, 7에 대응하는 배열을 보면 양쪽 모두 1로되어 있어서 리턴은 true로 던저질 것이다.

```
    +-+-+-+-+-+-+-+-+-+-+
    |0|0|1|1|0|0|0|1|0|0|
    +-+-+-+-+-+-+-+-+-+-+
     0 1 2 3 4 5 6 7 8 9
           *       *
```

그 다음 데이터 C를 검사한다면, 해시 함수를 거치면 3=h1(C), 6=h2(C)이다. 그런데 3은 1이나 6이 0이므로 false가 리턴되어 없는 것으로 간주하게 된다.

```
    +-+-+-+-+-+-+-+-+-+-+
    |0|0|1|1|0|0|0|1|0|0|
    +-+-+-+-+-+-+-+-+-+-+
     0 1 2 3 4 5 6 7 8 9
           *     *
```

이런 형태로 동작하는 방식이다. Bloom Filter는 기존 hash table의 key-value의 쌍으로 자정하지 않고 hash table상에 키가 존재하는지 안하는지 true, false 정보의 조각(Bit의 Set)이 저장된다고 보면 된다. 즉, 각 Value를 저장하는 대신에 Bloom Filter는 Key가 존재하는 지점(해시 함수에 의해)을 가르키는 Bits의 배열이라고 봐도 된다. 찾는 비트의 조합이 모두 1일 경우 제대로 찾을 확률이 높아진다는 것이다. Space와 Time을 적절하게 배합하고 고른 분포도를 가지며 Collision을 대처한다면 좋은 이득을 얻을 수 있다. 많은 비트를 할당할수록 성능은 좋을 수 있으나 많은 메모리가 필요하고 해싱 함수를 늘리게 되면 연산이 많아지게 되어 성능은 느리나 메모리를 덜 차지하게 되는 trade-off 관계가 존재한다. Optimizing하는게 관건이다.

## 2. Vector Clocks

[https://medium.com/@balrajasubbiah/lamport-clocks-and-vector-clocks-b713db1890d7](https://medium.com/@balrajasubbiah/lamport-clocks-and-vector-clocks-b713db1890d7)

## 3. Gossip Protocol

Decentralization, Partition tolerance를 지원하기 위해 Cassandra는 Gossip Protocol을 사용한다.
Gossip Protocol은 각 노드가 클러스터의 다른 노드의 상태정보를 추적하는 것이고 이는 매초 발생한다.

- Peer to peer communication protocol.
- Nodes are arranged in ring format.
- Data is replicated to multiple nodes.
- Nodes periodically exchange info. they have.
- Nodes also exchange their own info.
- Each message has its associated version.
- No master-slave concept, and hence no single point of failure.

### Gossip protocol 동작 방식

1. 초당 1 회, gossiper는 클러스터에서 임의의 노드를 선택하고 그 노드와 함께 gossip세션을 초기화합니다.
2. gossip initiator는 선택한 친구에게 GossipDigestSynMessage를 보냅니다.
3. 친구가 메시지를 받으면 GossipDigestAckMessage를 반환합니다.
4. initiator가 친구로부터 ack 메시지를 받으면 친구에게 GossipDigestAck2Message를 전송하여 gossip 라운드를 완료합니다
5. 만약 친구로 부터 ack를 받지 못하면 해당 노드를 convicts로 마크합니다.


## 4. Two Phase Commit

Two Phase Commit(a.k.a. 2PC)은 distributed system에서 atomic commit을 보장하는 프로토콜이다.

2PC에서 노드는 한 개의 coordinator와 여러 개의 cohort로 나누어진다. Coordinator는 commit 할 transaction을 만드는 노드고, cohort들은 coordinator가 보낸 transaction을 commit 하거나 revert 한다. 2PC는 이때 fail 하지 않은 모든 cohort가 같은 상태를 유지하도록 하는 것이다. 즉, fail 하지 않은 모든 노드는 다 같이 commit 하거나 revert 한다.

이때 coordinator를 어떻게 선정할지는 2PC의 영역이 아니다. 고정된 coordinator를 계속 사용할 수도 있고, 차례대로 돌아가면서 coordinator가 될 수도 있고, 별도의 leader election을 사용하여 coordinator를 선정할 수도 있다. 2PC는 어떻게든 coordinator가 선정된 뒤의 일이다.

2PC는 이름 그대로 2가지 phase로 나누어져 있다. 첫 번째 phase는 voting phase라고 부르고, 두 번째 phase는 commit phase라고 불린다. 각 phase의 시작은 coordinator가 보내는 메시지로 시작한다.

Voting phase에서 coordinator는 commit 하고 싶은 transaction을 commit 할지 말지 투표하는 요청을 cohort에게 보낸다. VOTE 메시지를 받은 cohort들은 이 transaction을 바로 commit 하지 않는다. 해당 transaction을 커밋할 수 있으면 YES 메시지를, 없으면 NO 메시지를 coordinator에게 보낸다.

Voting phase에서 coordinator가 quorum 이상의 YES 메시지나 NO 메시지를 모으면 commit phase를 시작한다. 이때 일부 cohort에 문제가 생겨서 더 진행되지 않는 것을 방지하기 위해서 일정 시간 응답을 주지 않는 cohort는 NO 메시지를 보냈다고 가정한다.

이때 quorum을 얼마로 잡는가에 따라서 시스템의 consistency model과 resilience가 결정된다. 예를 들어 N개의 coordinator가 있는 시스템에서 N개의 YES 메시지를 모아야 한다면, 하나의 failure도 용납하지 않는 resilient 하지 않지만, strong consistency를 보장하는 시스템이 된다. Quorum이 얼마가 되어야 하는지는 정해지지 않았다. 하지만 non-partition 상황에서 consistency를 보장하기 위해서는 최소 N/2 이상의 YES 메시지를 모아야 한다.

Coordinator가 quorum 이상의 YES 메시지를 받았으면 cohort들에게 COMMIT 메시지를 보내고, quorum 이상의 NO 메시지를 받았으면 cohort 들에게 ROLLBACK 메시지를 보낸다. cohort는 COMMIT 메시지를 받았으면 voting phase에서 받았던 transaction을 커밋하고, ROLLBACK 메시지를 받았으면 그 transaction을 버린다. COMMIT이든 ROLLBACK이든 메시지를 처리하고 나면 cohort는 coordinator에게 처리했다는 메시지를 보낸다. Coordinator가 cohort들에게 처리가 끝났다는 메시지를 받으면 commit phase가 끝난다.

위의 과정을 거쳐 2PC가 진행된다. 앞서 말했듯이 2PC는 괜찮은 resilience를 보이면서, 성능도 나쁘지 않기 때문에 많이 사용된다. 특히 atomic commit을 지원하는 프로토콜 중에서는 가장 적은 메시지 수로 commit 될 수 있는 프로토콜이다.

하지만 2PC에는 심각한 문제가 하나 있다. 2PC는 VOTE 메시지를 보낸 coordinator가 죽어서 COMMIT이나 ROLLBACK 메시지를 보내지 못하면 YES 메시지를 보낸 cohort가 안전하게 상태를 회복할 방법이 없다. 이는 YES 메시지를 보낸 cohort의 상태가 undefined이기 때문이다.

## 5. CAP

“분산 시스템에서는  3개 속성을 모두 가지는 것이 불가능하다!” 이다. 각 속성은 아래 3가지이다.

### Consistency (일관성)

- CAP이론에서 말하는 Consistency는 ACID의 ‘C’가 아니다! ACID의 ‘C’는 “데이터는 항상 일관성 있는 상태를 유지해야 하고 데이터의 조작 후에도 무결성을 해치지 말아야 한다”는 속성이다.
- CAP의 ‘C’는 “Single request/response operation sequence”의 속성을 나타낸다. 그 말은 쓰기 동작이 완료된 후 발생하는 읽기 동작은 마지막으로 쓰여진 데이터를 리턴해야 한다는 것을 의미한다.
- 모든 노드가 같은 시간에 같은 데이터를 보여줘야 한다. (저장된 데이터까지 모두 같을 필요는 없음)

### Availability (가용성)

- “특정 노드가 장애가 나도 서비스가 가능해야 한다”라는 의미를 가진다.
- 데이터 저장소에 대한 모든 동작(read, write 등)은 항상 성공적으로 리턴되어야 한다.
- 명확해 보이는 단어이기는 하지만 분산 시스템에서의 특징을 말하는 것이기 때문에 “서비스가 가능하다”와 “성공적으로 리턴”이라는 표현이 애매하다. 얼마동안 기다리는 것 까지를 성공적이라고 할 수 있느냐에 대한 문제가 남아있다. “20시간정도 기다렸더니 리턴이 왔어! Availability가 있는 시스템이야!”라고 할 수 없기 때문이다.

### Partitions Tolerance (분리 내구성)

- Tolerance to network Partitions인데 보통은 Partition-tolerance라고도 한다.
- 노드간에 통신 문제가 생겨서 메시지를 주고받지 못하는 상황이라도 동작해야 한다.
- Availablity와의 차이점은 Availability는 특정 노드가 “장애”가 발생한 상황에 대한 것이고 Tolerance to network Partitions는 노드의 상태는 정상이지만 네트워크 등의 문제로 서로간의 연결이 끊어진 상황에 대한 것이다

## 6. Eventual Consistency

서버가 여러대인 분산시스템에서 데이터를 조회했을 때 특정 서버는 변경된 데이터가 조회되고 일부는 변경되지 않은 상태로 조회될 수 있다. 그 때 데이터의 일관성을 위해서 모든 서버에 결과값을 질의하고 N개 이상이 같은 값을 반환할 때 사용자에게 해당 값을 보여주는 것을 의미한다.

ex > Cassandra에서 Consistency read level을 설정하는 것


## 7. Column Storage

### 정의

데이터의 저장을 칼럼단위로 처리하는 데이터베이스를 말한다. 

### 장점

칼럼 단위의 값은 데이터가 유사할 가능성이 높다. 이로 인해 높은 압축율을 얻을 수 있다. 

MIN, MAX, SUM, COUNT 와 같은 연산에서 높은 성능을 얻을 수 있다. 



### 종류

아마존 Redshift, 아파치 Cassandra, HBase 등이 있다. 

----

컬럼 지향 데이터베이스는 데이터를 컬럼 단위로 묶어서 저장한다. 그런 다음 이 컬럼값은 디스크 상에 연속적으로 저장된다. 

이 방식은 전통적인 데이터베이스의 전체 로우가 연속적으로 저장되는 일반적인 로우 지향형 접근방식과 다르다. 

컬럼 기반으로 데이터를 저장하는 이유는, 특정 쿼리에 대해서는 로우의 모든 데이터가 필요하지 않다는 가정에 기반하고 있다. 

이러한 경우는 특히 분석적인 데이터베이스에서 자주 발생하기 때문에 분석적 데이터베이스들은 이러한 형태의 저장 스키마를 사용하기 위한 좋은 후보이다. 

이렇듯 I/O 가 줄어든다는 이유만으로도 이 새로운 데이터 저장 구조를 채택할 법한데, 이 구조는 여기에 높은 압축률이라는 장점까지 제공한다. 

일반적으로 서로 다른 논리적인 로우 상의 같은 컬럼값들은 본질상 매우 유사하기 마련이고, 때로는 아주 약간씩만 다르기 때문에, 압축을 위해 서로 묶이는 편이 서로 상이한 값들로 이루어진 로우 지향 레코드 구조보다 훨씬 나을 때가 많다. 

대부분의 압축 알고리즘은 한정된 영역만을 바라보기 때문이다. 

컬럼에 기반하여 델타 압축이나 프리픽스 압축 같은 특화된 알고리즘을 선택하면 엄청나게 향상된 압축 비율을 얻을 수 있다. 

압축 비율이 좋으면 대역폭을 더 효율적으로 사용할 수 있다.

## 8. Consistent Hashing

메타 정보를 조회하지 않아도 클러스터에서 키가 저장된 노드를 바로 찾아갈 수 있는 방법이다. 

키의 해시 값을 구해서 키를 찾고, 이 해시 값만으로 노드를 찾아갈 수 있다. 

좀 더 자세히 살펴보면, 컨시스턴트 해싱은 일련의 해시 값을 가상의 링(ring)에 순서대로 나열했다고 가정하고 각 노드는 링에서 각자 맡은 범위만 처리하는 방법이다. 

만약 노드를 추가하면 특정 노드(많은 데이터를 가진 노드)가 맡고 있던 범위를 분할하여 새 노드에 할당한다. 

노드를 삭제할 때는 링에서 삭제된 노드가 맡고 있던 범위를 인접 노드가 맡는다. 

따라서 서비스 중에 노드의 추가/삭제로 인해 영향을 받는 노드 수를 최소화할 수 있다

## 9. Hinted Handoff

wirte해야할 서버가 장애로 죽었을 때, 해당 서버에 기록되어야할 데이터를 hint로그로 남겨두어 장애가 복구되면 해당 서버에 hint에 적어둔 데이터르 write해 consistency를 보장하기 위한 방법이다.

## 10. Lightweight Transactions and Paxos

독자적인 수정을 통해 읽기 및 쓰기 쿼리간에 다른 클라이언트가 들어오지 못하도록 보장하고자합니다.
Since the 2.0 release, Cassandra supports a lightweight transaction (or “LWT”) mechanism that provides linearizable consistency.
카산드라의 LWT 구현은 Paxos를 기반으로합니다.
Paxos는 마스터가 트랜잭션을 조정할 필요없이 분산 피어 노드가 제안서에 동의 할 수 있도록하는 합의 알고리즘입니다.
Paxos 및 기타 합의 알고리즘은 분산 트랜잭션에 대한 전통적인 2 단계 커밋 기반 접근 방식의 대안으로 부상했습니다

### Paxos 알고리즘 동작방식

기본적인 Paxos 알고리즘은 prepare / promise와 propose / accept의 두 단계로 구성됩니다.
데이터를 수정하기 위해 코디네이터 노드는 리더 역할을 수행하여 복제본 노드에 새 값을 제안 할 수 있습니다.
다른 노드는 다른 수정을 위해 동시에 리더 역할을 할 수 있습니다.
각 복제본 노드는 제안서를 확인하고 제안서가 최신 내용 인 경우 이전 제안서와 관련된 제안서를 수락하지 않습니다.
각 복제 노드는 아직 진행중인 마지막 제안을 반환합니다.
제안서가 대다수의 복제본에 의해 승인 된 경우, 리더는 제안서를 커미트하지만, 제안서를 선행 한 진행중인 제안서를 먼저 커밋해야한다는 경고가 표시됩니다.

1. Cassandra는 읽기전 쓰기를 사용하기 위해서 Paxos알고리즘에 두단계를 추가해서 사용합니다.
2. Prepare/Promise
3. Read/Results <--추가됨
4. Propose/Accept
5. Commit/Ack <--추가됨

따라서 성공적인 트랜잭션은 코디네이터 노드와 복제본 사이에 네 번의 라운드 트립이 필요합니다.
이것은 일반 쓰기보다 비용이 많이 들기 때문에 LWT를 사용하기 전에 유스 케이스에 대해 신중하게 생각해야합니다.

한줄 요약 : 강력한 일관성 보장하기 위해 사용 하지만 사용할 경우 비용이 많이 발생한다. 그래서 신중하게 생각해서 사용

## 11. Write Ahead 

Write Ahead Log는 분산 파일 시스템 위의 파일이다. WAL은 아직 permanent storage에 저장되지 않은 새로운 데이터를 저장하기 위해 쓰인다.

WAL을 사용하는 시스템에서 모든 수정은 적용 이전에 로그에 기록된다. 일반적으로 redo 및 undo 정보는 둘 다 로그에 저장된다.

한 예로 어느 프로그램이 특정 작업을 수행하는 동안 컴퓨터에 정전이 일어났다고 하자. 다시 시작할 때 프로그램은 어느 작업이 수행을 성공적으로 마쳤는지, 절반 성공했는지, 아니면 실패했는지를 잘 알고 있어야 한다. 로그 선행 기입이 사용된다면 프로그램은 이러한 로그를 검사하여 예기치 않은 정전 시 해야할 일과 실제로 했던 일을 비교하게 된다.

데이터 복구 로직을 만들 때 많이 사용

실제 데이터를 저장하기 전에 미리 저장하고 들어가니, 복구할때 WAL만 읽어서 순서데로 복구하면 됨

ex > HBase, Kafka에 사용됨

### Reference

- Bloomfilter([http://www.mimul.com/pebble/default/2012/03/30/1333089490367.html](http://www.mimul.com/pebble/default/2012/03/30/1333089490367.html))
- 2PC([https://swdev.tistory.com/2](https://swdev.tistory.com/2))
- 2PC([https://blog.seulgi.kim/2018/05/two-phase-commit.html](https://blog.seulgi.kim/2018/05/two-phase-commit.html))
- CAP([https://hamait.tistory.com/197](https://hamait.tistory.com/197))
- Column Storage([https://118k.tistory.com/400](https://118k.tistory.com/400))
- Consistent Hashing([https://charsyam.wordpress.com/2016/10/02/%EC%9E%85-%EA%B0%9C%EB%B0%9C-consistent-hashing-%EC%97%90-%EB%8C%80%ED%95%9C-%EA%B8%B0%EC%B4%88/](https://charsyam.wordpress.com/2016/10/02/%EC%9E%85-%EA%B0%9C%EB%B0%9C-consistent-hashing-%EC%97%90-%EB%8C%80%ED%95%9C-%EA%B8%B0%EC%B4%88/))
- Consistent Hashing([https://smallake.kr/?p=17730](https://smallake.kr/?p=17730))
