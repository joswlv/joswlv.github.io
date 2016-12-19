---
layout: post
title: GROUP_CONCAT
date: 2016-12-18 10:00
categories: MYSQL
---


# [MySQL] GROUP_CONCAT에 대해


`GROUP_CONCAT`은 서로 다른 결과를 한줄로 합쳐서 보여줘야 할 경우 사용하는 MySQL 명령어이다.

이는 전체 결과를 서버로 들고와서 for문을 돌려 문자열을 붙여도 되지만 SELECT쿼리를 사용하여 합쳐져 있는 문자열을 받는게 더 편하다.


```sql
select * from test;
```

|key|value|
|----|-----|
|가구|편한가구|
|가구|싼 가구|
|가구|신혼집 가구|
|가구|신혼부부 가구|
|가구|신혼부부 가구|

* 이런 결과를 `GROUP_CONCAT`을 사용하면 다음과 같다.

```sql
SELECT key, GROUP_CONCAT(value) FROM test GROUP BY key;
```

|key|value|
|----|----|
|가구|편한가구,싼 가구,신혼집 가구,신혼부부 가구, 신혼부부 가구|

* `GROUP_CONCAT`의 기본적으로 문자열 사이에 쉼표(,)가 붙게 된다. 구분자를 변경하고 싶을 때에는 `SEPARATOR '구분자'`를 함께 사용하면 된다.

```sql
SELECT kye, GROUP_CONCAT(value SEPARATOR '/') FROM test GROUP BY key;
```

|key|value|
|----|----|
|가구|편한가구/싼 가구/신혼집 가구/신혼부부 가구/ 신혼부부 가구|

* 합쳐지는 문자열에 중복되는 문자열을 제거 할때는 `DISTINCT`를 사용한다.

```sql
SELECT key, GROUP_CONCAT(DISTINCT value) FROM test GROUP BY key;
```

|key|value|
|----|----|
|가구|편한가구,싼 가구,신혼집 가구,신혼부부 가구|

* 문자열을 정렬하여 나타내고 싶으면 `order by`를 사용한다.


```sql
SELECT key, GROUP_CONCAT(DISTINCT value ORDER BY valeu) FROM test GROUP BY key;
```

### 정리

MySQL에서 group by 로 문자열을 합칠땐 group_concat 을 이용한다.

1. 기본형 : group_concat(필드명)
2. 구분자 변경 : group_concat(필드명 separator '구분자')
3. 중복제거 : group_concat(distinct 필드명)
4. 문자열 정렬 : group_concat(필드명 order by 필드명)


### 참고


Name|	Description|
|----|-----|
|AVG()|	Return the average value of the argument|
|BIT_AND()|	Return bitwise AND|
|BIT_OR()	|Return bitwise OR|
|BIT_XOR()|	Return bitwise XOR|
|COUNT()	|Return a count of the number of rows returned|
|COUNT(DISTINCT)	|Return the count of a number of different values|
|GROUP_CONCAT()|	Return a concatenated string|
|MAX()|	Return the maximum value|
|MIN()|	Return the minimum value|
|STD()|	Return the population standard deviation|
|STDDEV()|	Return the population standard deviation|
|STDDEV_POP()|	Return the population standard deviation|
|STDDEV_SAMP()|	Return the sample standard deviation|
|SUM()|	Return the sum|
|VAR_POP()	|Return the population standard variance|
|VAR_SAMP()|	Return the sample variance|
|VARIANCE()|	Return the population standard variance|

표 : [MySQL_Doc](http://dev.mysql.com/doc/refman/5.7/en/group-by-functions.html#function_avg)


#### [tip]

* 오라클의 경우 mysql의 group_concat을 아래와 같이 대체하여 사용이 가능하다.
* 오라클 10G : WM_CONCAT()
* 오라클 11G : LISTAGG()



### Reference

* [과일가게 개발자](http://fruitdev.tistory.com/16)
* [음머어's 까망별](http://blackbull.tistory.com/3)
* [초보개발자 이야기](http://ra2kstar.tistory.com/56)