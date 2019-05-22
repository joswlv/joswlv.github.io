---
layout: post
title: Spark SQL jdbc사용시 주의할 사항
date: 2019-05-20 13:10
categories: Spark
---
# Spark SQL jdbc사용시 주의할 사항

## 개요

Spark로 작업한 결과를 Mysql로 적재하는 일이 있었다.

자주 이용하는 방식은 Mysql Connection관리와 key 중복이 발생할때 update를 하기 위해서 아래 두가지 방식을 많이 사용했다. 

- `collect()`하여 드라이버노드에서 insert 하는 방식
- RDD로 변형하여`foreachPartition`를 돌면서 insert 하는 방식

하지만 데이터가 매우 커지면 위 방식들은 OOM이나, IOException을 발생 시켰다.

그래서 Dataset의 jdbc write 방식을 이용하게 되었다.
이를 이용하면서 몇가지 문제점을 기록하고자 한다.

## jdbc write 사용방법

```scala
val prop = new Properties()
prop.setProperty("driver", "com.mysql.jdbc.Driver")
prop.setProperty("user", "mysql.user")
prop.setProperty("password", "mysql.password")

resultDataset.write.mode(SaveMode.Append)
	.jdbc("Mysql.url", "tableName", prop)
```

이렇게만 하면 각 executor에서 insert문을 실행한다.

## 주의점

`SaveMode`에 따라서 Mysql table을 다루는 방식이 다른다.

- SaveMode.Overwrite
	- 관련 table을 drop - create를 한뒤 insert를 진행한다.
	- 2.1버전 이상부터는 truncate옵션을 줘 drop table를 방지할 수 있다.(default는 false, 사용법 : `.option("truncate", true)`)
- SaveMode.Append
	- insert가 진행되는데, key겹쳐면 insert를 진행함 하지만 mysql에서 에러를 던짐
- SaveMode.Ignore
	- insert가 진행되는데, key겹쳐면 isnert를 진행안함 mysql까지 안감 
- SaveMode.ErrorIfExists
	- default option
	- insert가 진행되는데, key겹쳐면 isnert를 진행안함 mysql까지 안감, spark에서 에러를 던짐

사용하는 점에서는 문제가 없지만 해결할 방법이 없는 경우가 있다.

```sql
INSERT INTO table (column_list)
VALUES (value_list)
ON DUPLICATE KEY UPDATE
   c1 = v1, 
   c2 = v2,
   ...;
```
위 Query처럼 key겹칠때 update를 하고 싶을 때가 있다. 
하지만 SparkSql에서는 해당 기능을 지원하지 않는다.

그래서 다른 방법을 사용해야한다.(파일에 쓰고 insert 등)

또하나 주의할 점은
`SaveMode.Overwrite`에서 cretae 될때 테이블 정보가 없기 때문에 Dataset에 있는 colum정보로 table이 생성된다. 

그렇기 때문에 `SaveMode.Overwrite`를 사용할때는 `truncate`옵션을 true로 사용하는 것이 좀 더 안전하다.


### Reference

- [Tips for using JDBC in Apache Spark SQL](https://medium.com/@radek.strnad/tips-for-using-jdbc-in-apache-spark-sql-396ea7b2e3d3)
- [jdbc Spark PR](https://github.com/apache/spark/pull/14086)