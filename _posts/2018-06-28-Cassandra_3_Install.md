---
layout: post
title: Cassandra 3.x Install
date: 2018-06-28 15:10
categories: Cassandra
---

# Cassandra 3.x Install


이번에 회사에서 C* 2.0에서 3.x로 버전업을 하였다. 이에 설치방법을 정리해본다.


### 서버정보 

- OS : CentOS Linux release 7.4.1708 (Core)
- Disk : RAID 1+0 (/ 20GB, /home1 40GB, SWAP 2GB, /data 나머지)
- MEM : 32 GB

## 1. SSD Optimization설정

> 참고 링크 : [https://docs.datastax.com/en/dse/5.1/dse-admin/datastax_enterprise/config/configRecommendedSettings.html](https://docs.datastax.com/en/dse/5.1/dse-admin/datastax_enterprise/config/configRecommendedSettings.html)

### 1-1. disk I/O Scheduler, rotaional, read ahead 설정

```bash
$ sudo chmod u+x /etc/rc.local
$ sudo systemctl start rc-local
```

```bash
$ sudo vi /etc/rc.local
 
$ echo noop > /sys/block/sda/queue/scheduler
$ echo 0 > /sys/class/block/sda/queue/rotational
$ echo 8 > /sys/class/block/sda/queue/read_ahead_kb
```

```bash
설정 값 설명) disk i/o scheduler - noop 으로 설정
sysFS 회전 플래그 - false(0) 으로 설정
readahead 블록 장치 값을 8kb 로 설정
```

### 1-2 SSD Trim 기능 활성화

```bash
$ sudo sudo vi /etc/fstab
 
UUID=d0241535-bee1-4798-a0dc-678157eea087 /data                 xfs     discard,noatime         0 0
```


> 설정 값 설명) 트림(TRIM)은 SSD 디스크를 사용하면서 운영체제에서 삭제한 쓰레기 파일을 SSD 디스크 자체에서도 삭제시켜서 SSD가 사용하면서 느려지는 것을 개선해주는 기능


## 2. Disable zone_reclaim_mode on NUMA systems

> 
참고링크 : [https://m.blog.naver.com/PostView.nhn?blogId=503109&logNo=220820781338&proxyReferer=https%3A%2F%2Fwww.google.com%2F](https://m.blog.naver.com/PostView.nhn?blogId=503109&logNo=220820781338&proxyReferer=https%3A%2F%2Fwww.google.com%2F)

```bash
$ sudo yum install numactl
$ numactl --show
$ numactl -H
$ nuastat -cm
$ cat /proc/buddyinfo
```

> 요약 : NUMA - 메모리 접근의 대기시간을 줄이기 위해서 CPU Processor 마다 특정 범위의 메모리를 할당해서 지역성을 부여하는 방식 / vm.zone_reclaim_mode - 특정 영역의 메모리가 부족할 경우 다른 영역의 메모리를 할당하는데 zone 안에서 재할당하지 않음(다른 zone 에서 가져와서 사용함)


```bash
$ sudo vi /etc/rc.local
 
$ echo 0 > /proc/sys/vm/zone_reclaim_mode
```

## 3. vm mmap count config

> 참고링크 : [https://m.blog.naver.com/PostView.nhn?blogId=nix102guri&logNo=90156339946&proxyReferer=https%3A%2F%2Fwww.google.com%2F](https://m.blog.naver.com/PostView.nhn?blogId=nix102guri&logNo=90156339946&proxyReferer=https%3A%2F%2Fwww.google.com%2F)

```bash

$ sudo vi /etc/sysctl.conf
 
$ vm.max_map_count = 1048575
```

> 요약 : max_map_count 는 특정 프로세스가 소유할 수 있는 VMA (가상 메모리 영역) 수의 제한을 허용하는 것이다. 프로그램이 메모리에 맵핑을 시도할 때 파일 공유, 공유 메모리에 대한 세그먼트에 대한 링크 또는 힙 공간을 할당 할 때 프로세스의 수명동안 생성됩니다. 이 값을 조정하면 프로세스가 소유할 수 있는 VMA의 양이 제한됩니다. 프로세스가 소유할 수 있는 VMA의 양을 제한하는 것이다. 이를 제한하면 프로세스가 VMA 제한에 도달했을 때 메모리 부족 오류가 발생한다.


## 4. JDK Install

> 주의!!
> OpenJDK, OracleJDK 1.8.161/1.8.162 버전에서는 RMI 에러가 발생하여 카산드라가 실행이 되지 않는다.

jdk rpm파일을 다운로드하여 각 노드에 설치한다.

```bash
$ sudo yum localinstall -y jdk-8u151-linux-x64.rpm
```

## 5. JNA Install

> 참고링크 : [http://knight76.tistory.com/entry/jni-vs-jna](http://knight76.tistory.com/entry/jni-vs-jna)


```bash
$ sudo yum install -y jna
```

> 요약 : Cassandra는 최근에 JNI(Java Native Interface) 대신 JNA (Java Native Access) 방식을 이용해서 native 영역에 메모리를 copy 하는 작업이 많은 경우에 사용되어 플랫폼의 속도를 향상시키는 방법을 쓰고 있다.


## 6. Jemalloc Install

> 참고링크 : http://channy.creation.net/project/dev.kthcorp.com/2011/05/12/last-free-lunch-facebooks-memory-allocator-jemalloc/index.html
http://www.databasetorque.com/2017/09/jemalloc-and-cassandra.html

```bash
$ wget http://dl.fedoraproject.org/pub/epel/7/x86_64/Packages/j/jemalloc-3.6.0-1.el7.x86_64.rpm

$ sudo yum localinstall -y jemalloc-3.6.0-1.el7.x86_64.rpm
```

> 요약 : facebook 이 소개한 메모리 할당자(malloc) 로서 즉, 시스템에서 메모리를 받아오는 역할이다. 최근 멀티코어, 멀티스레드에 동작하는 프로그램에서는 속도, 공간효율성의 측면에서 굉장히 중요하다.
> 
> Cassandra에서 Jemalloc 를 활성화하면 Java heap 공간을 줄이게 된다. Cassandra 에 쓰여진 데이터는 먼저 Heap 메모리에 있는 MemTable 에 저장되고, MemTable 이 가득차면 SStable 로 Flush 된다. JVM GC는 Heap Memory를 지우는데 사용된다. 때때로 이러한 수집 프로세스로 인해 가비지 수집 일시 중지가 발생하여 Cassandra 에 문제가 발생하기도 한다.

malloc : 동적으로 메모리를 할당하는 함수 (힙 영역에 메모리를 할당)


## 7. Cassandra Install

### 7-1 Cassandra  yum repo추가

```bash
sudo vi /etc/yum.repos.d/cassandra.repo
 
[cassandra]
name=Apache Cassandra
baseurl=https://www.apache.org/dist/cassandra/redhat/311x/
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://www.apache.org/dist/cassandra/KEYS
```

### 7-2 Cassandra Install

```bash
sudo yum -y install cassandra
```

### 7-3 Cassandra Config 수정

```bash

vi /etc/cassandra/conf/cassandra.yaml
 
10 cluster_name: 'testCassandra'
25 num_token: 256
73 hints_directory: /data/cassandra/hints
103 authenticator: PasswordAuthenticator
190 data_file_directories:
191     - /data/cassandra/data
196 commitlog_directory: /data/cassandra/commitlog
368 saved_caches_directory: /data/cassandra/saved_caches
425     - seeds: 시드 노드 IP
438 concurrent_reads: 128
439 concurrent_writes: 256
440 concurrent_counter_writes: 128
444 concurrent_materialized_view_writes: 128
466 disk_optimization_strategy: ssd
539 memtable_flush_writers: 128
575 trickle_fsync: true
599 listen_address: # 각 노드별 IP
676 rpc_address: # 각 노드별 IP
811 concurrent_compactors: 32
819 compaction_throughput_mb_per_sec: 32
1074 internode_compression: all
1094 enable_user_defined_functions: true
```

### 7-4 Cassandra directory 생성

```bash
$ sudo mkdir -p /data/cassandra
$ sudo chown cassandra:cassandra /data/cassandra
```

### 7-5 Cassandra Start

```bash
$ sudo systemctl start cassandra
$ sudo systemctl enable cassandra
```

### 7-6 Cassandra Cluster Check

```bash
nodetool status
nodetool describecluster
```

UN => UP / NORMAL
Cluster Name => testCassandra
Snitch => SimpleSnitch
Partitioner => Murmur3Partitioner

## 8. Cassandra User 설정

### 8-1 cqlsh 접속

```bash
$ cqlsh -u cassandra -p cassandra
```

### 8-2 신규 superuser생성

```sql
CREATE USER testuser WITH PASSWORD 'testuser123' SUPERUSER;
```

### 8-3 신규 superuser로 접속하여 cassandra 계정 변경

```sql
ALTER USER cassandra WITH PASSWORD 'newpasswd' NOSUPERUSER;
```

### 8-4 사용자 변경확인

```sql
LIST USER;
```

### 8-5 system_auth keyspace의 replication 설정 수정

```sql

ALTER KEYSPACE system_auth WITH REPLICATION = {'class' : 'SimpleStrategy', 'replication_factor' : 3};
```

## 9. JMX 권한 설정

### 9-1 JMX 설정 변경

```bash
$ sudo vi /etc/cassandra/conf/cassandra-env.sh
 
243 if [ "x$LOCAL_JMX" = "x" ]; then
244     LOCAL_JMX=no
```

### 9-2 User 및 Password설정

```bash
$ sudo cp /usr/java/jdk1.8.0_151/jre/lib/management/jmxremote.password.template /etc/cassandra/jmxremote.password

$ sudo vi /etc/cassandra/jmxremote.password
 
testuser testuser123
 
$ sudo chown cassandra:cassandra /etc/cassandra/jmxremote.password
$ sudo chmod 400 /etc/cassandra/jmxremote.password
```

### 9-3 JMX 접근 확인

```bash
nodetool -u testuser -pw testuser123 status
```
