---
layout: post
title: Cassandra CqlBulkOutputFormat사용법
date: 2019-01-13 15:10
categories: Cassandra
---

# Cassandra CqlBulkOutputFormat사용법

## 도입배경

- Cassandra에서는 Spark connector, cqlsh 등 다양한 방법의 데이터 업로드 방법을 제공하나 대부분 insert query를 만들어 Cassandra에 실행하는 형태임.
- 많은 수의 query가 발생할 경우 Network I/O 부하가 심해지고, Cassandra의 Memory 사용량 증가와 잦은 compaction으로 인해 성능 저하가 발생함.
- 이를 방지하기 위해 Cassandra는 외부 서버에서 SSTable을 만든 후 올리는 SSTableLoader라는 Tool을 제공함.
- 그러나, 외부 서버에서 SSTable 생성시 많은 Memory 사용과 CPU 점유률로 인해 부하 분산을 고민하게 되었음.

## 분산 환경에서 Cassandra bulk 업로드 방법들

1. CqlBulkOutputFormat, CQL3 based, easier to program, requires C* V2+ and JDK 7 in Hadoop environment
2. BulkOutputFormat, thrift based, less abstraction and needs more low-level work, provided in C* V1 and OK with JDK 6
3. CqlSSTableWriter + SSTableLoader, CQL3 based, easier to program, but no managed parallelism, requires C* V2 but no need for Hadoop environment, multi-thread/parallel computing mechanism would required for scalability

[출처: [https://shenghuawan.wordpress.com/2015/01/20/cassandra-bulk-loading-summary/](https://shenghuawan.wordpress.com/2015/01/20/cassandra-bulk-loading-summary/)]

--> 3번은 분산 환경에서 적합하지 않음

--> 2번은 레퍼런스는 많으나 Cassandra 버전업 이후 deprecated 됨

--> 1번 방법인 CqlBulkOutputFormat을 이용해 개발하기로 결정함

## CqlBulkOutputFormat 소스

[https://github.com/apache/cassandra/blob/trunk/src/java/org/apache/cassandra/hadoop/cql3/CqlBulkOutputFormat.java](https://github.com/apache/cassandra/blob/trunk/src/java/org/apache/cassandra/hadoop/cql3/CqlBulkOutputFormat.java)

## CqlBulkOutputFormat 기본 동작

- Mapper 혹은 Reducer에서 넘어온 `List<ByteBuffer>` 데이터를 입력으로 받음
- 전달된 데이터들은 buffer size대로 쪼개져 여러개의 SSTable 파일로 생성됨
- 생성된 SSTable들은 지정된 대역폭대로 압축되어 Cassandra에 전송됨

## Mapper 구현

- CqlBulkOutputFormat은 `List<ByteBuffer>`로만 입력을 받으므로 Mapper(혹은 Reducer)를 Java로 구현하는 게 필요함
- Cassandra 테이블 컬럼의 데이터형에 따라 ByteBuffer로 casting 해줘야 하는데 Cassandra API 중 org.apache.cassandra.utils.ByteBufferUtil 을 사용하면 편리함

```java
public void map(LongWritable key, Text data, 
             OutputCollector<Text, List<ByteBuffer>> output,
             Reporter reporter) throws IOException {
    
 String[] dataArr = data.toString().split(delimiter);
 int categoryId = Integer.parseInt(dataArr[0]);
 String uid = dataArr[1].toLowerCase();
 SimpleDateFormat sdf = new SimpleDateFormat(dateformat);
 long dtSeconds = 0;
 try {
     Date dt = sdf.parse(uploadData);
     dtSeconds = dt.getTime();
 } catch (java.text.ParseException e) { }
    
 // make column data
 List<ByteBuffer> columns = new ArrayList<ByteBuffer>();
 columns.add(ByteBufferUtil.bytes(uid));
 columns.add(ByteBufferUtil.bytes(dtSeconds));
 columns.add(ByteBufferUtil.bytes(score));
 columns.add(ByteBufferUtil.bytes(categoryId));
    
 output.collect(new Text(uid), columns);
 }
```

## CqlBulkOutputFormat 옵션

Hadoop Streaming -D 옵션으로 직접 설정하거나, -conf 옵션에 properties xml 파일로 지정함

- cassandra.output.keyspace : keyspace 이름 설정
- mapreduce.output.basename : table 이름 설정
- cassandra.columnfamily.schema.user_interest_category : table schema

ex) CREATE TABLE dmp.user_interest_category_test ( uid text, date timestamp, score float, category_id int, PRIMARY KEY (uid, date, score, category_id) )

- cassandra.columnfamily.insert.user_interest_category : insert query

ex) INSERT INTO dmp.user_interest_category_test (uid, date, score, category_id) VALUES (?, ?, ?, ?);

- cassandra.output.keyspace.username : cassandra user name
- cassandra.output.keyspace.passwd : password
- cassandra.output.thrift.address : cassandra 서버 ip 목록 (comma seperate)
- cassandra.output.partitioner.class : org.apache.cassandra.dht.Murmur3Partitioner
- mapreduce.job.user.classpath.first
- hadoop core 라이브러리와 cassandra 라이브러리 간에 충돌이 발생할 경우가 있음. guava api의 경우 사용하는 버전이 달라서 에러가 발생하므로 cassandra의 라이브러리를 우선 사용하도록 이 옵션을 true로 설정
- mapreduce.output.bulkoutputformat.buffersize
- sstable 생성시 사용하는 buffer size (기본 64MB).
- 설정값을 크게 할수록 한번에 생성되는 sstable size가 커지는 대신에 메모리 사용량이 많아서 MR 작업시 java.lang.OutOfMemoryError: GC overhead limit exceeded 에러가 발생하기도 함
- 너무 적게 설정할 경우 sstable 개수가 많아져 cassandra의 compaction이 자주 일어날 가능성이 있음
- mapreduce.output.bulkoutputformat.streamthrottlembits
- cassandra에 sstable 전송시 설정되는 bandwidth 값. (기본 0: 무한대)
- 설정하지 않을 경우 굉장히 빠른 속도로 cassandra 서버에 전송되나 cassandra 서버의 network inbound 대역폭을 전부 점유해버려 일시적으로 cassandra 서버에 큰 부하를 줄 수 있음.
- 상황에 따라 적절한 값으로 설정하는 게 중요함.

## Maven build

```xml
<repositories>
	<repository>
		<id>ngnentRepository</id>
		<name>NMP Repository</name>
		<url><http://nexus.nhnent.com/content/groups/public></url>
	</repository>
	<repository>
		<id>mvnRepository</id>
		<name>mvn Repository</name>
		<url><https://mvnrepository.com/artifact/org.apache.cassandra/cassandra-all></url>
	</repository>
</repositories>
    
<dependencies>
	<dependency>
		<groupId>org.apache.hadoop</groupId>
		<artifactId>hadoop-common</artifactId>
		<scope>system</scope>
		<systemPath>${HADOOP_COMMON}</systemPath>
		<version>2.0.0</version>
	</dependency>
	<dependency>
		<groupId>org.apache.hadoop</groupId>
		<artifactId>hadoop-core</artifactId>
		<scope>system</scope>
		<systemPath>${HADOOP_MAPREDUCE_CORE}</systemPath>
		<version>0.20</version>
	</dependency>
	<dependency>
		<groupId>org.apache.cassandra</groupId>
		<artifactId>cassandra-all</artifactId>
		<version>2.1.15</version>
	</dependency>
</dependencies>
```

## Hadoop Streaming MapReduce 작업 실행

Maven 빌드한 jar 파일을 포함 시키고, 작업한 Mapper class와 outputformat을 명시한다.

```shell
hadoop jar ${HADOOP_STREAMING_JAR} \\
-D mapred.job.name="dmp-interest-bulkload-"${DATE} \\
-D mapred.map.tasks=2 \\
-D mapreduce.job.user.classpath.first=true \\
-D cassandra.output.keyspace.username=${CASSANDRA_USER} \\
-D cassandra.output.keyspace.passwd=${CASSANDRA_PAWD} \\
-D cassandra.output.thrift.address=${CASSANDRA_ADDR} \\
-D cassandra.output.partitioner.class=org.apache.cassandra.dht.Murmur3Partitioner \\
-conf ${CONF_DIR}/schema_user_interest_category_app.xml \\
-libjars ${JAR_PATH} \\
-input ${INPUT_PATH} \\
-output ${OUTPUT_PATH} \\
-mapper com.toast.exchange.dmp.interest.bulkload.AppAdidBulkload \\
-reducer NONE \\
-outputformat org.apache.cassandra.hadoop.cql3.CqlBulkOutputFormat
```

## Hadoop Streaming MapReduce 작업 로그

각 Map (혹은 Reduce)의 stdout 로그를 살펴보면 sstable 파일들이 생성되고, 전송된 결과를 확인할 수 있다.
MapReduce작업이 실행된 data node를 접근해 생성된 sstable 파일들을 확인해볼 수도 있다.

sstable 생성로그<br/>

```shell
09:25:46.685 [Thread-11] DEBUG o.apache.cassandra.io.util.FileUtils - Renaming /data8/yarn/nm/usercache/irteam/appcache/application_1456906327100_79670/container_1456906327100_79670_01_000004/tmp/dmp/user_interest_category-c485093c-ef8b-4d06-b870-9465cd076a3b/dmp-user_interest_category-tmp-ka-1-Digest.sha1 to /data8/yarn/nm/usercache/irteam/appcache/application_1456906327100_79670/container_1456906327100_79670_01_000004/tmp/dmp/user_interest_category-c485093c-ef8b-4d06-b870-9465cd076a3b/dmp-user_interest_category-ka-1-Digest.sha1
09:25:46.687 [Thread-11] DEBUG o.apache.cassandra.io.util.FileUtils - Renaming /data8/yarn/nm/usercache/irteam/appcache/application_1456906327100_79670/container_1456906327100_79670_01_000004/tmp/dmp/user_interest_category-c485093c-ef8b-4d06-b870-9465cd076a3b/dmp-user_interest_category-tmp-ka-1-TOC.txt to /data8/yarn/nm/usercache/irteam/appcache/application_1456906327100_79670/container_1456906327100_79670_01_000004/tmp/dmp/user_interest_category-c485093c-ef8b-4d06-b870-9465cd076a3b/dmp-user_interest_category-ka-1-TOC.txt
09:25:46.687 [Thread-11] DEBUG o.apache.cassandra.io.util.FileUtils - Renaming /data8/yarn/nm/usercache/irteam/appcache/application_1456906327100_79670/container_1456906327100_79670_01_000004/tmp/dmp/user_interest_category-c485093c-ef8b-4d06-b870-9465cd076a3b/dmp-user_interest_category-tmp-ka-1-Index.db to /data8/yarn/nm/usercache/irteam/appcache/application_1456906327100_79670/container_1456906327100_79670_01_000004/tmp/dmp/user_interest_category-c485093c-ef8b-4d06-b870-9465cd076a3b/dmp-user_interest_category-ka-1-Index.db
09:25:46.688 [Thread-11] DEBUG o.apache.cassandra.io.util.FileUtils - Renaming /data8/yarn/nm/usercache/irteam/appcache/application_1456906327100_79670/container_1456906327100_79670_01_000004/tmp/dmp/user_interest_category-c485093c-ef8b-4d06-b870-9465cd076a3b/dmp-user_interest_category-tmp-ka-1-Statistics.db to /data8/yarn/nm/usercache/irteam/appcache/application_1456906327100_79670/container_1456906327100_79670_01_000004/tmp/dmp/user_interest_category-c485093c-ef8b-4d06-b870-9465cd076a3b/dmp-user_interest_category-ka-1-Statistics.db
09:25:46.688 [Thread-11] DEBUG o.apache.cassandra.io.util.FileUtils - Renaming /data8/yarn/nm/usercache/irteam/appcache/application_1456906327100_79670/container_1456906327100_79670_01_000004/tmp/dmp/user_interest_category-c485093c-ef8b-4d06-b870-9465cd076a3b/dmp-user_interest_category-tmp-ka-1-Filter.db to /data8/yarn/nm/usercache/irteam/appcache/application_1456906327100_79670/container_1456906327100_79670_01_000004/tmp/dmp/user_interest_category-c485093c-ef8b-4d06-b870-9465cd076a3b/dmp-user_interest_category-ka-1-Filter.db
09:25:46.688 [Thread-11] DEBUG o.apache.cassandra.io.util.FileUtils - Renaming /data8/yarn/nm/usercache/irteam/appcache/application_1456906327100_79670/container_1456906327100_79670_01_000004/tmp/dmp/user_interest_category-c485093c-ef8b-4d06-b870-9465cd076a3b/dmp-user_interest_category-tmp-ka-1-CompressionInfo.db to /data8/yarn/nm/usercache/irteam/appcache/application_1456906327100_79670/container_1456906327100_79670_01_000004/tmp/dmp/user_interest_category-c485093c-ef8b-4d06-b870-9465cd076a3b/dmp-user_interest_category-ka-1-CompressionInfo.db
09:25:46.689 [Thread-11] DEBUG o.apache.cassandra.io.util.FileUtils - Renaming /data8/yarn/nm/usercache/irteam/appcache/application_1456906327100_79670/container_1456906327100_79670_01_000004/tmp/dmp/user_interest_category-c485093c-ef8b-4d06-b870-9465cd076a3b/dmp-user_interest_category-tmp-ka-1-Data.db to /data8/yarn/nm/usercache/irteam/appcache/application_1456906327100_79670/container_1456906327100_79670_01_000004/tmp/dmp/user_interest_category-c485093c-ef8b-4d06-b870-9465cd076a3b/dmp-user_interest_category-ka-1-Data.db
```

sstable 전송로그<br/>

```shell
09:25:48.236 [StreamConnectionEstablisher:3] INFO  o.a.c.streaming.StreamCoordinator - [Stream #5082a580-6023-11e6-a70e-5fc5e5829401, ID#0] Beginning stream session with /x.x.x.x
09:25:48.235 [STREAM-OUT-/x.x.x.x] DEBUG o.a.c.streaming.ConnectionHandler - [Stream #5082a580-6023-11e6-a70e-5fc5e5829401] Sending File (Header (cfId: e5ec2a50-3fd2-11e5-9d62-af38b156d6d1, #0, version: ka, estimated keys: 24064, transfer size: 2436481, compressed?: true, repairedAt: 0), file: /data8/yarn/nm/usercache/irteam/appcache/application_1456906327100_79670/container_1456906327100_79670_01_000004/tmp/dmp/user_interest_category-c485093c-ef8b-4d06-b870-9465cd076a3b/dmp-user_interest_category-ka-1-Data.db)
09:25:48.235 [STREAM-OUT-/x.x.x.x] DEBUG o.a.c.streaming.ConnectionHandler - [Stream #5082a580-6023-11e6-a70e-5fc5e5829401] Sending File (Header (cfId: e5ec2a50-3fd2-11e5-9d62-af38b156d6d1, #0, version: ka, estimated keys: 22528, transfer size: 2483732, compressed?: true, repairedAt: 0), file: /data8/yarn/nm/usercache/irteam/appcache/application_1456906327100_79670/container_1456906327100_79670_01_000004/tmp/dmp/user_interest_category-c485093c-ef8b-4d06-b870-9465cd076a3b/dmp-user_interest_category-ka-1-Data.db)
09:25:48.236 [STREAM-OUT-/x.x.x.x] DEBUG o.a.c.streaming.ConnectionHandler - [Stream #5082a580-6023-11e6-a70e-5fc5e5829401] Sending File (Header (cfId: e5ec2a50-3fd2-11e5-9d62-af38b156d6d1, #0, version: ka, estimated keys: 22912, transfer size: 2574224, compressed?: true, repairedAt: 0), file: /data8/yarn/nm/usercache/irteam/appcache/application_1456906327100_79670/container_1456906327100_79670_01_000004/tmp/dmp/user_interest_category-c485093c-ef8b-4d06-b870-9465cd076a3b/dmp-user_interest_category-ka-1-Data.db)
09:25:48.315 [STREAM-OUT-/x.x.x.x] DEBUG o.a.c.s.c.CompressedStreamWriter - [Stream #5082a580-6023-11e6-a70e-5fc5e5829401] Start streaming file /data8/yarn/nm/usercache/irteam/appcache/application_1456906327100_79670/container_1456906327100_79670_01_000004/tmp/dmp/user_interest_category-c485093c-ef8b-4d06-b870-9465cd076a3b/dmp-user_interest_category-ka-1-Data.db to /10.161.26.40, repairedAt = 0, totalSize = 2459515
09:25:48.324 [STREAM-OUT-/x.x.x.x] DEBUG o.a.c.s.c.CompressedStreamWriter - [Stream #5082a580-6023-11e6-a70e-5fc5e5829401] Start streaming file /data8/yarn/nm/usercache/irteam/appcache/application_1456906327100_79670/container_1456906327100_79670_01_000004/tmp/dmp/user_interest_category-c485093c-ef8b-4d06-b870-9465cd076a3b/dmp-user_interest_category-ka-1-Data.db to /10.161.26.156, repairedAt = 0, totalSize = 2379097
09:25:48.325 [STREAM-OUT-/x.x.x.x] DEBUG o.a.c.s.c.CompressedStreamWriter - [Stream #5082a580-6023-11e6-a70e-5fc5e5829401] Start streaming file /data8/yarn/nm/usercache/irteam/appcache/application_1456906327100_79670/container_1456906327100_79670_01_000004/tmp/dmp/user_interest_category-c485093c-ef8b-4d06-b870-9465cd076a3b/dmp-user_interest_category-ka-1-Data.db to /10.161.26.37, repairedAt = 0, totalSize = 2482971
09:25:48.328 [STREAM-OUT-/x.x.x.x] DEBUG o.a.c.s.c.CompressedStreamWriter - [Stream #5082a580-6023-11e6-a70e-5fc5e5829401] Start streaming file /data8/yarn/nm/usercache/irteam/appcache/application_1456906327100_79670/container_1456906327100_79670_01_000004/tmp/dmp/user_interest_category-c485093c-ef8b-4d06-b870-9465cd076a3b/dmp-user_interest_category-ka-1-Data.db to /10.161.26.39, repairedAt = 0, totalSize = 2424141
09:25:48.333 [STREAM-OUT-/x.x.x.x] DEBUG o.a.c.s.c.CompressedStreamWriter - [Stream #5082a580-6023-11e6-a70e-5fc5e5829401] Start streaming file /data8/yarn/nm/usercache/irteam/appcache/application_1456906327100_79670/container_1456906327100_79670_01_000004/tmp/dmp/user_interest_category-c485093c-ef8b-4d06-b870-9465cd076a3b/dmp-user_interest_category-ka-1-Data.db to /10.161.26.21, repairedAt = 0, totalSize = 2436481
09:25:48.336 [STREAM-OUT-/x.x.x.x] DEBUG o.a.c.s.c.CompressedStreamWriter - [Stream #5082a580-6023-11e6-a70e-5fc5e5829401] Start streaming file /data8/yarn/nm/usercache/irteam/appcache/application_1456906327100_79670/container_1456906327100_79670_01_000004/tmp/dmp/user_interest_category-c485093c-ef8b-4d06-b870-9465cd076a3b/dmp-user_interest_category-ka-1-Data.db to /10.161.26.38, repairedAt = 0, totalSize = 2574224
09:25:48.342 [STREAM-OUT-/x.x.x.x] DEBUG o.a.c.s.c.CompressedStreamWriter - [Stream #5082a580-6023-11e6-a70e-5fc5e5829401] Start streaming file /data8/yarn/nm/usercache/irteam/appcache/application_1456906327100_79670/container_1456906327100_79670_01_000004/tmp/dmp/user_interest_category-c485093c-ef8b-4d06-b870-9465cd076a3b/dmp-user_interest_category-ka-1-Data.db to /10.161.26.157, repairedAt = 0, totalSize = 2483732
09:25:48.350 [STREAM-OUT-/x.x.x.x] DEBUG o.a.c.s.c.CompressedStreamWriter - [Stream #5082a580-6023-11e6-a70e-5fc5e5829401] Start streaming file /data8/yarn/nm/usercache/irteam/appcache/application_1456906327100_79670/container_1456906327100_79670_01_000004/tmp/dmp/user_interest_category-c485093c-ef8b-4d06-b870-9465cd076a3b/dmp-user_interest_category-ka-1-Data.db to /10.161.26.22, repairedAt = 0, totalSize = 2461039
09:27:24.853 [STREAM-OUT-/x.x.x.x] DEBUG o.a.c.s.c.CompressedStreamWriter - [Stream #5082a580-6023-11e6-a70e-5fc5e5829401] Finished streaming file /data8/yarn/nm/usercache/irteam/appcache/application_1456906327100_79670/container_1456906327100_79670_01_000004/tmp/dmp/user_interest_category-c485093c-ef8b-4d06-b870-9465cd076a3b/dmp-user_interest_category-ka-1-Data.db to /10.161.26.38, bytesTransferred = 2574224, totalSize = 2574224
09:27:24.863 [STREAM-IN-/x.x.x.x] DEBUG o.a.c.streaming.ConnectionHandler - [Stream #5082a580-6023-11e6-a70e-5fc5e5829401] Received Received (e5ec2a50-3fd2-11e5-9d62-af38b156d6d1, #0)
09:27:24.864 [STREAM-OUT-/x.x.x.x] DEBUG o.a.c.streaming.ConnectionHandler - [Stream #5082a580-6023-11e6-a70e-5fc5e5829401] Sending Complete
09:27:24.876 [STREAM-IN-/x.x.x.x] DEBUG o.a.c.streaming.ConnectionHandler - [Stream #5082a580-6023-11e6-a70e-5fc5e5829401] Received Complete
09:27:24.877 [STREAM-IN-/x.x.x.x] DEBUG o.a.c.streaming.ConnectionHandler - [Stream #5082a580-6023-11e6-a70e-5fc5e5829401] Closing stream connection handler on /x.x.x.x
09:27:24.880 [STREAM-IN-/x.x.x.x] INFO  o.a.c.streaming.StreamResultFuture - [Stream #5082a580-6023-11e6-a70e-5fc5e5829401] Session with /x.x.x.x is complete
```

## CqlBulkOutputFormat bulk 업로드 성능 측정

- 대상 데이터 : 4.5억 row수의 13G 데이터
- buffer size와 대역폭의 변화를 주고 성능 측정함
- buffer size : 64MB(기본값), 대역폭 : 0 (기본값)
    - 결과 : sstable 생성 10분, 전송 5분
- buffer size : 96MB, 대역폭 지정: 1MBps
    - 결과 : sstable 생성 10분, 전송 1시간 50분
