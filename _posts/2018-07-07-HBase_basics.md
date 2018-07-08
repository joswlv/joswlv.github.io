---
layout: post
title: HBase의 이해
date: 2018-07-07 15:10
categories: HBase
---

# HBase의 이해

## 요점

1. 모든 컬럼 패밀리는 같은 같은 file system에 저장됨.
2. turning 와 storage specification이 column family level에서 일어나므로, 모든 column family 멤버는 같은 general 접근 패턴과 사이즈 특징을 가진다.
3. Data 는 region으로 나눠진다. region은 table이 나눠진 단위이다. region서버에 나눠진 단위이다.
4. 데이터는 HDFS에 저장된다.
5. server-master architecture 이다.
6. region server 는  data의 read/write를 담당한다.
7. master 는 create, delete tables 을 담당한다.
8. zookeeper는 상태 유지를 담당한다.
9. hadoop datanode에 region 서버가 유지하는 데이터를 저장한다.
10. hadoop namenode에는 metadata information을 저장한다.
11. datanode에는 region server가 n개 있다.

____

> 아래 글은 [https://mapr.com/blog/in-depth-look-hbase-architecture/](https://mapr.com/blog/in-depth-look-hbase-architecture/) 글을 정리 한것입니다.

### <HBase Architecture>

HBase는 물리적으로는 Master-Slave 구조로 3가지 서버의 구성 요소를 가집니다.

1. HMaster : 테이블의 생성과 삭제, Region할당
2. Region Server : 데이터의 일고 쓰기를 담당한다.
3. Zookeeper : HDFS의 한 부분인 zookeeper는 cluster의 상태를 유지한다.
	
Region Server가 담당하는 데이터들은 Hadoop의 DataNode에 저장된다.

모든 HBase Data는 HDFS 파일안에 저장된다.

Region Server는 HDFS DataNode와 협력을 함으로써, 데이터 지역성을 가능하게 한다.

NameNode는 물리적인 데이터 블럭에 대한 metadata 정보를 유지한다.

![https://mapr.com/blog/in-depth-look-hbase-architecture/assets/blogimages/HBaseArchitecture-Blog-Fig1.png](https://mapr.com/blog/in-depth-look-hbase-architecture/assets/blogimages/HBaseArchitecture-Blog-Fig1.png)

### HBase HMaster 

zookeeper는 분산 시스템의 멤버들에게 상태정보를 공유하기 위해서 사용되는 서비스이다.

Region 서버와 Active HMaster는 session을 이용해서 zookeeper에 연결한다.

zookeeper는 ephemeral node를 유지한다.

각각의 Resion Serversms ephemeral node를 하나 생성한다.

HMaster는 이 노드를 모니터해서 사용가능한 region server를 발견하고, 또 server의 failure도 모니터한다.

HMaster는 ephemeral node하나 생성하는 것을 경쟁한다.

zookeeper는 첫번째 것을 고려하고, 하나의 master가 active할 수 있게 한다.

Actvie Master는 하트비트를 zookeeper에게 보내고, inactive HMaster는 active HMaster의 실패의 알림을 듣는 역할을 한다.

만약 Region server나 active HMaster가 하트비트 보내는 것을 실패하면, session이 만료되고, 대응하는 노드는 삭제된다.

Active HMaster는 region server를 듣고, 실패시 region server의 복구할 것이고, Inactive HMaster는 active HMaster의 실패를 듣고 있다가, active HMaster가 실패하면 inactive HMaster가 active가 된다.

>정리
>
> 1) Active HMaster 2) Inactive HMaster 3) Region Server 이렇게 있는데, HMaster와 Regsion Server는 ephemeral node를 만들어서 세션을 zookeeper와 연결한다. 그 노드를 통해서, ActiveHMaster는 Region Server의 상태를 체크하고, 어떤 서버가 이용가능한지 보고, 그리고 Region server가 실패하면 복구를 명령한다. Inactive HMaster는 ActiveHMaster를 감시한다. Active HMaster가 실패하면, 자기 자신이 Active가 된다.

![https://mapr.com/blog/in-depth-look-hbase-architecture/assets/blogimages/HBaseArchitecture-Blog-Fig5.png](https://mapr.com/blog/in-depth-look-hbase-architecture/assets/blogimages/HBaseArchitecture-Blog-Fig5.png)


### HBase First Read or Write

HBase에는 Meta Table이라는 특별한 Catalog Table이 있다. Cluster내에 있는 region의 위치를 담고 있다.

zookeeper는 region의 위치를 Meta Table에 저장한다.

**처음으로 읽고 쓰는 작업**

1. Client는 zookeeper로 부터 Meta Table을 관리하는 region server를 갖고 온다.
2. Client는 접근하기 원하는 row key에 대응하는 region server를 갖고 오기 위해서 meta server에게 쿼리를 날린다.
3. 대응하는 region server로 부터 row를 얻게 된다.

나중에 다시 읽을 때는, client는 meta location을 가지고 오기 위해 캐쉬를 이용한다. (이전에 미리 읽어놨기 때문에 캐쉬에 저장되어 있다.)

region이 다른 곳으로 이동되어서, 정보를 가져오는 것에 실패하지 않는 한, Meta Table에 쿼리를 할 필요가 없다. (정보를 가져오는 것을 실패 했을 때는 re-query를 하고, 캐쉬를 update한다.)

![https://mapr.com/blog/in-depth-look-hbase-architecture/assets/blogimages/HBaseArchitecture-Blog-Fig6.png](https://mapr.com/blog/in-depth-look-hbase-architecture/assets/blogimages/HBaseArchitecture-Blog-Fig6.png)

### HBase Meta Table

- Meta Table은 시스템내의 모든 regions의 리스트를 유지하는 HBase Table이다.
- `.META` table은 B-Tree같은 구조이다.
- `.META` table 구조는 다음과 같다.
	- key : region start key, region id
	- value : region server

>정리
>
>Meta Table은 시작되는 row key정보를 갖고 있고, region id를 갖고 있으며, 어느 server지를 가르키는 region server를 value로 갖고 있다.

![https://mapr.com/blog/in-depth-look-hbase-architecture/assets/blogimages/HBaseArchitecture-Blog-Fig7.png](https://mapr.com/blog/in-depth-look-hbase-architecture/assets/blogimages/HBaseArchitecture-Blog-Fig7.png)

### Region Server Components

Region Server는 HDFS Data Node 위에서 실행되고 다음 component를 가진다.

- WAL : Write Ahead Log는 분산 파일 시스템 위의 파일이다. WAL은 아직 permanent storage에 저장되지 않은 새로운 데이터를 저장하기 위해 쓰인다.
- BlockCache : Read Cache인데, 자주 읽는 데이터를 메모리에 저장한다. Least Recently Used Data is evicted when full.(LRU)
- MemStore : Write Cache인데, Disk에 아직 쓰이지 않는 새로운 데이터를 저장한다. 하나의 region에 하나의 column family마다 하나의 memstore이 있다.
- Hfiles : 디스크상에 정렬된 key-value로써의 row를 저장한다.

![https://mapr.com/blog/in-depth-look-hbase-architecture/assets/blogimages/HBaseArchitecture-Blog-Fig8.png](https://mapr.com/blog/in-depth-look-hbase-architecture/assets/blogimages/HBaseArchitecture-Blog-Fig8.png)

### HBase Write Steps

#### HBase Write 1단계

client가 put request를 날리면, 첫번째로 WAL이 일어난다.

Edits가 disk에 저장되어 있는 WAL파일 끝에 로그를 쓴다.

WAL는 Server Crash가 발생시 not-yet-persisted data를 복구하기 위해 사용된곤한다.

![https://mapr.com/blog/in-depth-look-hbase-architecture/assets/blogimages/HBaseArchitecture-Blog-Fig9.png](https://mapr.com/blog/in-depth-look-hbase-architecture/assets/blogimages/HBaseArchitecture-Blog-Fig9.png)

#### HBase Write 2단계

일단 데이터가 WAL에 쓰여지지만, MemStore에 놓여진다. 그런후에, Put Request는 client에게 Acknowledgement 반환할 것을 요청한다.

![https://mapr.com/blog/in-depth-look-hbase-architecture/assets/blogimages/HBaseArchitecture-Blog-Fig10.png](https://mapr.com/blog/in-depth-look-hbase-architecture/assets/blogimages/HBaseArchitecture-Blog-Fig10.png)

MemStore는 정렬된 KeyValue로써, 메모리안에 업데이트를 저장한다. HFile에 저장되어 진것과 같은 모양이다.

column family당 하나의 MemStore이 있다. Update는 column family마다 정렬되어 진다.

![https://mapr.com/blog/in-depth-look-hbase-architecture/assets/blogimages/HBaseArchitecture-Blog-Fig11.png](https://mapr.com/blog/in-depth-look-hbase-architecture/assets/blogimages/HBaseArchitecture-Blog-Fig11.png)

#### HBase Write 3단계

MemStore에 충분한 데이터를 축적하면 저장되어진 데이터가 HDFS내에 새로운 HFile로 쓰여진다.

HBase는 하나의 column family당 여러개의 HFile을 사용하고, 실제의 cell 또는 Key-Value객체를 담고 있다.

MemStore안에서 Key-Value로 정려로디어진채로 생성되어진 이들 파일은 디스크에 파일로써 flush된다.

하나의 CF마다 하나의 MemStore가 있다. 하나가 full이 되면 모든것이 flush가 된다.
또 마지막으로 쓰여진 스퀀스 번호를 저장하여, 시스템이 지금까지 저장된 것이 무엇인지 알수 있게 한다.
이것이 왜 column family 수의 제한이 있는지 이유이다.

가장 큰 시퀀스 숫자는 각각의 HFile안에 meta filed로써 저장 되어 진다. 지속하고 있는것이 끝나지고 어디서 계속되는지 반영하기 위해서 meta file에 저장한다.

![https://mapr.com/blog/in-depth-look-hbase-architecture/assets/blogimages/HBaseArchitecture-Blog-Fig12.png](https://mapr.com/blog/in-depth-look-hbase-architecture/assets/blogimages/HBaseArchitecture-Blog-Fig12.png)

### HBase HFile

데이터는 정렬된 key-value를 담고있는 HFile로 저장된다. MemStore에 적당한 데이터를 모우면 전체 정려로딘 key-value set이 새로운 Hfile로 HDFS안에 쓰여진다.
이것은 연속적인 쓰기작업이다. Disk Drive Head 움직임을 피하기 때문에 매우 빠르다.

![https://mapr.com/blog/in-depth-look-hbase-architecture/assets/blogimages/HBaseArchitecture-Blog-Fig13.png](https://mapr.com/blog/in-depth-look-hbase-architecture/assets/blogimages/HBaseArchitecture-Blog-Fig13.png)

#### HFile 구조

하나의 HFile은 Multi-Layered Index를 가지고, 이것 때문에 전체 파일을 읽을 필요없이 HBase내에 데이터를 찾을 수 있게 해준다. (Multi-level index는 b+tree와 같다.)

- key-value 쌍은 증가하는 순서로 저장된다.
- row key에 의해 indexed point는 64KB의 block안에 key-value를 가진다.
- 각각의 block의 마지막 key는 중간의 index안에 놓여진다.
- 각 block의 마지막 key는 중간의 index안에 놓여진다.
- root index point는 중간 index를 가르킨다.

Trailer는 meta block을 가리킨다. 그리고 이것은 file에 지속적인 데이터의 끝에 쓰여진다.
또 Trailer는 bloom filter와 같은 정보와 시간 범위 정보를 가진다. (bloom filter는 특정한 row key를 포함하지 않은 파일을 read skip을 해주어 read비용을 줄여준다.)
시간 범위 정보는 읽고자 하는 시간 범위가 아닐경우 파일을 skipping하는데 유용하다.

![https://mapr.com/blog/in-depth-look-hbase-architecture/assets/blogimages/HBaseArchitecture-Blog-Fig14.png](https://mapr.com/blog/in-depth-look-hbase-architecture/assets/blogimages/HBaseArchitecture-Blog-Fig14.png)

### HBase Read Merge

**읽는 순서**

1. 스캐너는 Block cache에서 Row Cell을 찾는다.
2. 스캐너는 MemStore에서 찾는다.
3. 만약 스캐너가 RowCell을 못 찾았다면, 그 위에 HBase는 해당 Row Cell이 들어 있는 HFile을 memory로 로드하기 위해서 Block cache와 Bloom Filter을 사용한다.

![https://mapr.com/blog/in-depth-look-hbase-architecture/assets/blogimages/HBaseArchitecture-Blog-Fig16.png](https://mapr.com/blog/in-depth-look-hbase-architecture/assets/blogimages/HBaseArchitecture-Blog-Fig16.png)

MemStore마다 많은 HFile이 존재한다. 이 의미는 한번 읽을때 마다, 여러개의 파일이 검사되어진다는 의미이고, 이것은 성능에 영향을 줄 가능성이 크다

이것을 `Read Amplification`이라고 부른다.


### HBase Minor Compaction

HBase는 자동적으로 어떤 작은양의 HFile을 집어와서, 약간 더 큰 HFile안에 다시 쓴다. 이 과정을 `Minor Compaction`이라고 한다.

Minor Compaction은 Storage File 숫자를 줄이고 작은 HFile을 좀 더 큰 HFile로 쓴다. 이 작업은 Merge Sort로 진행된다.

![https://mapr.com/blog/in-depth-look-hbase-architecture/assets/blogimages/HBaseArchitecture-Blog-Fig18.png](https://mapr.com/blog/in-depth-look-hbase-architecture/assets/blogimages/HBaseArchitecture-Blog-Fig18.png)

### HBase Major Compaction

Major Compaction은 한 Region안에 있는 모든 Hfile을 하나의 Column Family마다의 HFile로 머지하고 다시 쓰는 작업이다. 그리고 이 과정에서, 셀을 drop, 삭제하거나 expire한다.

이 작업이 이뤄지면, 읽기 성능을 향상시킨다. 그러나 Major Compaction이 모든 파일을 새로 쓰기 때문에 Disk I/O와 Network Traffic이 많이 발생한다. 이 작업을 `write amplification`이라고 한다.

Major Compaction은 자동적으로 실행되도록 스케줄되어 있다. write amplification 때문에 Major Compaction은 보통 주말 또는 오후에 일어난다. (Note that MapR-DB has made improvements and does not need to do compactions)

![https://mapr.com/blog/in-depth-look-hbase-architecture/assets/blogimages/HBaseArchitecture-Blog-Fig19.png](https://mapr.com/blog/in-depth-look-hbase-architecture/assets/blogimages/HBaseArchitecture-Blog-Fig19.png)

### Region = Contiguous Keys

Table은 하나또는 여러개의 Region으로 나눠진다. Region은 start key와 end key사이에 연속적으로 정렬된 범위의 row를 갖는다. default로 각각의 Region은 1G 사이즈를 갖는다.
각 Region은 RegionServer에 의해 client에게 서비스 제공한다.
Region server은 1000개 정도의 Region을 관리한다. 

![https://mapr.com/blog/in-depth-look-hbase-architecture/assets/blogimages/HBaseArchitecture-Blog-Fig20.png](https://mapr.com/blog/in-depth-look-hbase-architecture/assets/blogimages/HBaseArchitecture-Blog-Fig20.png)

### Region Split

초기에는 하나의 테이블에 하나의 Region이 있다. Region이 점점 커질수로, 이것은 두개의 child Region으로 나눠진다.
두개의 child Region이 같은 Region Server에서 평행하여 열리고, split 가 HMaster에게 보고한다. Load balancing 이유 때문에, HMaster는 새로운 region을 다른 서버로 보내도록 스케줄링 할 수도 있다.

![https://mapr.com/blog/in-depth-look-hbase-architecture/assets/blogimages/HBaseArchitecture-Blog-Fig21.png](https://mapr.com/blog/in-depth-look-hbase-architecture/assets/blogimages/HBaseArchitecture-Blog-Fig21.png)

### Read Load Balancing

하나의 Region이 커져서,  split되어서 두개의 Region 으로 나눠지고, load balancing 문제 때문에 하나의 Region이 다른 서버로 옮겨지게 된다.
이것은, major compaction이 데이터 file을 Region 서버의 local node로 옮길때까지, 새로운 Region server serving으로 끝난다. 

HBase data는 처음 쓸 때는, local이지만, region이 다른 서버로 옮겨지면, 더이상 local이 아니다. ( 정확히는 major compaction이 일어날 때까지)

![https://mapr.com/blog/in-depth-look-hbase-architecture/assets/blogimages/HBaseArchitecture-Blog-Fig22.png](https://mapr.com/blog/in-depth-look-hbase-architecture/assets/blogimages/HBaseArchitecture-Blog-Fig22.png)

이 말인즉, 처음에 Region 이 분리 되어지면, 다른 Region Server가 해당 Region을 맡게 된다.
이때는 Remote 로 데이터를 맡는 것이다.
이후에 Major compaction이 일어나면 그때는 local이 된다.

### HDFS Data Replication

모든 쓰가와 읽그는 primary node로부터 이고, primary node 까지이다. HDFS는 WAL와 HFile block을 복제한다. 
HFile block replication은 자동적으로 이루어진다. Hbase은 HDFS에 의존하여 이것의 파일을 저장하듯이 하여 데이터 안전을 제공한다. 
데이터가 HDFS에 쓰여지면 하나의 copy는 locally하게 저장되고, 그런 후에 replicated가 secondary node로 된다. 세번째 copy는 tertiary node에 진행된다.

![https://mapr.com/blog/in-depth-look-hbase-architecture/assets/blogimages/HBaseArchitecture-Blog-Fig23.png](https://mapr.com/blog/in-depth-look-hbase-architecture/assets/blogimages/HBaseArchitecture-Blog-Fig23.png)

### HBase Crash Recovery

zookeeper는 region server의 heart beat를 잃게 되면  Node failure를 고려할것이다. HMaster은 Region Server가 실패되어진다는 알림을 받을 것이다.

HMaster는 region server가 crash 되었다는것을 알아차리고, HMaster는 region을  충돌한 서버로부터 active Region server로 재할당 할것이다. 
충돌난 region server에서 아직 disk에 쓰여지지 않은 memstore edit를 복구 하기 위해서, HMaster는 충돌난 서버에 존재하는 WAL을 여러개의 file로 나눌 것이고, 이 파일을 새로운 region server의 data node에 저장한다. 
각각의 Region server은 그런뒤에 WAL을 replay 해서, memstore을 rebuild 한다. 

![https://mapr.com/blog/in-depth-look-hbase-architecture/assets/blogimages/HBaseArchitecture-Blog-Fig25.png](https://mapr.com/blog/in-depth-look-hbase-architecture/assets/blogimages/HBaseArchitecture-Blog-Fig25.png)


### Data Recovery

WAL 파일들은 많은 작업의 리스트를 가진다. 하나의 edit는 하나의 put/delete를 나타낸다. Edits는 시간순서로 일어나고, 저장을 위해서 추가적인것은 디스크에 저장되는 WAL 파일의 끝에 시간순서로 일어난다.

그러니깐, 만약 data가 여전히 memory에 있고, HFile로 저장되어진게 아닌채로 실패가 발생하면 무슨일이 일어날까?

WAL이 replayed 되어진다. 추가와 정렬은 current MemStore에 이뤄진다.
끝으로 MemStore은 HFile에 change본이 flush 된다.

![https://mapr.com/blog/in-depth-look-hbase-architecture/assets/blogimages/HBaseArchitecture-Blog-Fig26.png](https://mapr.com/blog/in-depth-look-hbase-architecture/assets/blogimages/HBaseArchitecture-Blog-Fig26.png)


### HBase 장단점

##### 장점

1. 강력한 일관성 모델
	- 쓰고 읽는 작업 동안 동일한 value를 볼 수 있다. 
2. 자동화된 확장
	- data가 너무 커지면, region은 자동으로 split된다.
	- HDFS를 사용하여 데이터를 뿌리고 복제한다. 
3. 빌트인 복구
	- Write Ahead Log를 사용한다. 
4. Hadoop과의 통합
	- MapReduce 가 쉽다. 

##### 단점

- WAL replay가 느리다.
- slow complex crash recovery
- Major Compaction I/O storms 

