---
layout: post
title: RDBMS vs NOSQL
date: 2017-02-13 10:04
categories: BigData
---

#### RDBMS? NOSQL?

- 데이터의 읽기 쓰기 등 퍼포먼스에 치중한다면 NOSQL, 트랜잭션과 같은 정합성 위주의 시스템을 사용한다면 RDBMS
- RDBMS 컬럼 변경 용이하지 않음, NOSQL 컬럼 변경 용이
- NOSQL의 경우 sorting, join, grouping, range query, index 매우 취약
- RDBMS 학습 비용 x
- NOSQL 학습 비용 소요 (운영시 어떤 장애상황이 생길지 예측이 어려움)
- NOSQL 가장 큰 장점 (Scale-Out, RDBMS보다 상대적으로 빠른 쓰기/읽기)

<br>

*NOSQL 분류*

\[ 키 밸류형 \] redis, memcached, Oracle Coherence

\[ 컬럼형   \] Cassandra, HBASE, Cloud Datastore

\[ 문서형   \] MongoDB, Couchbase, MarkLogic, PostgreSQL, MySQL, DynamoDB MS-DocumentDB

\[ 그래프형  \] Neo4j

<br>

| *DataStore* | *설 명* | *장 점* | *단 점* |
|-----------|-----------|-----------------|------------------------|
| Cassandra | - Facebook에 의해 2008년 아파치 오픈소스로 공개된 분산 데이터 베이스 (자바 언어 기반)<br> - 컬럼 단위로 관리되어 컬럼형으로 분류<br> - 대용량의 데이터 트랜잭션에 대해 고성능 처리가 가능(실제 트위터 MYSQL -> Cassandra로 전환)                                                                                                                                                                         | - 대량으로 쓰기가 발생하는 서비스에 좋음<br> - 확장성이 뛰어남<br> - Apache Foundation에서 개발중이며커뮤니티 활발<br> - Scale-Out                                                                                                                                                                                  | - 최소 3대 이상 구성(클러스터 환경)<br> - 복잡한 조건 검색 불가<br> - 데이터 갱신 및 입력시 Atomic한 처리가 힘듬                 |
| HBase     | - 대량 데이터를 우수한 성능으로 데이터 일관성을 보장하면서 다뤄야 할 때 주로 사용<br> - 대량 데이터 분석 및 처리를 위해 사용되는 Hadoop의 산하 프로젝트로 시작된 데이터베이스 (HDFS 및 MapReduce등과 함께 사용하기에 최적화) <br> - 수십 테러바이트가 넘는 빅데이터에 적합                                                                                                                               | - 하둡 기반에서 동작하고 다양한 하둡 의 도구들과 상호 운영성이 좋음<br> - 데이터 일관성 보장 우수(상대적)                                                                                                                                                                                                             | - 5대 미만에서는 사용할 수 없다(대규모 전용)<br> - 성능이 좋진 않다 (상대적)                                                      |
| MongoDB   | - MongoDB는 10gen 사에서 개발된 높은 성능과 확장성을 가지고 있는 데이터베이스<br> - NoSQL 데이터베이스에서는 문서형 데이터베이스로 분류(C언어 기반)<br> - 데이터를 입력할때 데이터 구조 정보를 포함하여 BSON(JSON을 바이너리화한것)형식으로 저장하고, key value로 사용<br> - NON-SCHEMA<br> -비정형 데이터, 파일 데이터등의 스키마프리(Scheme free)모델에서 적합<br> - SQL 과 비슷한 방식의 쿼리 사용 | - 스키마 없이 사용 가능 <br> - SQL 과 비슷한 방식의 쿼리 사용<br> - 몽고는 쓰기할때 메모리에 먼저 Write 후에 1분 단위로 Flushing하는 Write back 방식을 사용한기 때문에 write성능이 좋음<br> - Read시에는 파일의 Index를 메모리에 로딩해   놓고 찾는다(memory mapped file) <br> - 빠름<br> - 다양한 기능 제공<br> | - JOIN이나 트랜잭션 처리가 불가능<br> - 디스크에 쓰기가 비동기식으로 이루어진다. 때문에 경우에 따라 데이터가 유실될 가능성도 있다. |

#### \[ Cassandra & HBase \]

- 카산드라 클러스터 설정 및 구성이 HBase 클러스터 구성보다 훨씬 쉽다.
- 카산드라가 일반적으로 write시 5배 이상의 더 나은 성능, read시 4배 이상의 성능을 보인다.

#### \[ Cassandra & MongoDB \]

- Cassandra 노드가 추가될수록 MonogoDB 보다 훨씬 나은 선형적인 성능 향상을 보인다.
- 다중 Index가 필요한 구조라면 MongoDB를 선택하고, 데이터 항목 변경이 많고 unique access가 많은 경우라면 Cassandra가 적합
- [http://db-engines.com/en/system/Cassandra;MongoDB](http://db-engines.com/en/system/Cassandra;MongoDB)

#### 성능비교

* [https://academy.datastax.com/planet-cassandra/nosql-performance-benchmarks](https://academy.datastax.com/planet-cassandra/nosql-performance-benchmarks)
* [https://www.datastax.com/nosql-databases/benchmarks-cassandra-vs-mongodb-vs-hbase](https://www.datastax.com/nosql-databases/benchmarks-cassandra-vs-mongodb-vs-hbase)
