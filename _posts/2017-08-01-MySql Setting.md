---
layout: post
title: MySQL 기본세팅법
date: 2017-08-01 10:04
categories: MYSQL
---

# MySql Setting

### mac에 mysql를 설치할때

```
brew install mysql

mysql.server start
```

### 사용할 DB만들기

DB를 만들기 전에 MySql 기본 캐리터셋을 설정해야한다.

MySQL에서는 설정 파일을 다음 순서데로 읽는다.

- /etc/my.cnf
- /etc/mysql/my.cnf
- /usr/local/mysql/etc/my.cnf
- ~/.my.cnf

그래서 `/etc/my.cnf`파일 있는지 확인 하고 없으면 `cp /usr/local/mysql/support-files/my-default.cnf /etc/my.cnf`으로 생성하고 다음내용을 추가한다.

```bash
[mysqld]
character-set-server=utf8
collation-server=utf8_general_ci

init_connect=SET collation_connection=utf8_general_ci
init_connect=SET NAMES utf8

[client]
default-character-set=utf8

[mysql]
default-character-set=utf8
```

그리고 서버 재시작

### root 페스워드 설정

```
[root@test ~]# /usr/bin/mysql_secure_installation
```


### DB생성 및 사용자 추가하기

```bash
mysql> create database testdb;
Query OK, 1 row affected (0.00 sec)

mysql> show create database testdb;
+----------+------------------------------------------------------------------+
| Database | Create Database                                                  |
+----------+------------------------------------------------------------------+
| testdb   | CREATE DATABASE `testdb` /*!40100 DEFAULT CHARACTER SET utf-8 */ |
+----------+------------------------------------------------------------------+
1 row in set (0.00 sec)
```

캐릭터 셋이 utf-8이 되어 있다.

유저생성은 다음과 같다.

```bash
// localhost로 접속만 허용
mysql> grant all privileges on testdb.* to dev@localhost identified by 'password123';

// 원격 접속도 허용
mysql> grant all privileges on testdb.* to dev@'%' identified by 'password123';
```

```bash
$> mysql -u dev -p
Enter password: (앞에서 지정한 패스워드 입력)
... 중략 ...
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| testdb             |
+--------------------+
3 rows in set (0.00 sec)
```


### 타임존 설정

`/etc/localtime` 파일 설정을 한국시간으로 바꾸기만 하면 된다.

```bash
$> ln -sf /usr/share/zoneinfo/Asia/Seoul /etc/localtime
$> ls -al | grep /etc/locatime
lrwxrwxrwx  1 root root      30  7월 31 12:17 localtime -> /usr/share/zoneinfo/Asia/Seoul
$> date
2017. 07. 31. (월) 12:18:08 KST
```

### 부팅시 자동시작 등록

```
[root@test ~]# chkconfig --list mysqld
mysqld         	0:off	1:off	2:off	3:off	4:off	5:off	6:off
[root@test ~]# chkconfig mysqld on
[root@test ~]# chkconfig --list mysqld 
mysqld         	0:off	1:off	2:on	3:on	4:on	5:on	6:off
```

### Refrence

- [https://goo.gl/8eEoyJ](https://goo.gl/8eEoyJ)