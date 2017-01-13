---
layout: post
title: Stored Procedure으로 Insert
date: 2017-01-09 08:04
categories: MYSQL
---


# MySQL Stored Procedure으로 Insert


쿼리테스트를 위한 테스트 데이터 1000만건을 insert을 해야한다. 이런 미션을 해결하기 위해 procedure를 사용했다.

## 1. 저장 프로시저란?

여러 SQL문을 하나의 SQL문 처럼 정리하여 `CALL procedure명`라는 명령어로 실행할 수 있게 만든 것을 저장프로시저(Stored Procedure)라고 한다. 

저장프로시저는 MySQL버전 5.0이상에서 사용할 수 있다.

## 2. 첫번째 시도! (no using TRANSACTION)

```sql
DELIMITER $$
DROP PROCEDURE IF EXISTS FILL_RATE_TEST_DATA$$
CREATE PROCEDURE myTEST()
BEGIN
	DECLARE i INT DEFAULT 1;
	DECLARE j INT DEFAULT 1;
	DECLARE log_date VARCHAR(255);	
		
	WHILE i <= 30 DO
		SET log_date = DATE_FORMAT(DATE_SUB(NOW(), INTERVAL i DAY), '%Y%m%d');
		WHILE j <= 30000 DO
			INSERT INTO 테이블 (LOG_DATE, ID, REQ_CNT) VALUES (log_date, '0x2011', 3000000);
			SET j = j + 1;
		END WHILE;
		SET j = 0;
	END WHILE;
END$$
DELIMITER $$

CALL myTEST()
```

> `DELIMITER`는 저장프로시저에서 구분문자를 `세미콜론(;)`이 아닌 다른 문자로 변경해준다. 위의 예에서는 `$$`을 사용했다.

90만건을 insert를 하는 데 `236s`을 기다려야한다.

## 3. 두번째 시도(using TRANSACTION)

다음으로 시도한 방버은 쿼리실행을 `TRANSCATION`안에 넣는 것이다.

```sql
DELIMITER $$
DROP PROCEDURE IF EXISTS FILL_RATE_TEST_DATA$$
CREATE PROCEDURE myTEST()
BEGIN
	DECLARE i INT DEFAULT 1;
	DECLARE j INT DEFAULT 1;
	DECLARE log_date VARCHAR(255);	
	START TRANSACTION;	
	WHILE i <= 30 DO
		SET log_date = DATE_FORMAT(DATE_SUB(NOW(), INTERVAL i DAY), '%Y%m%d');
		WHILE j <= 30000 DO
			INSERT INTO 테이블 (LOG_DATE, ID, REQ_CNT) VALUES (log_date, '0x2011', 3000000);
			SET j = j + 1;
		END WHILE;
		SET j = 0;
	END WHILE;
	COMMIT;
END$$
DELIMITER $$

CALL myTEST()
```

`START TRANSACTION`과 `COMMIT;`사이에 쿼리문을 넣어주면 된다.

실행속도는 `68s` 거의 4배 빨라진 것을 알 수 있다.

## 3. 더 나아가기

Procedure에서 cursor와 trigger 등을 응용 사용하는 법이 있다. 

다음 사이트를 참조하면 된다.

* [저장 프로시저 활용하기](http://recoveryman.tistory.com/186)
* [프로시저/커서사용](http://yookeun.github.io/database/2015/04/10/mysql-procedure/)
* [프로시저/커서사용2](http://bizadmin.tistory.com/entry/MySQL-Fetch-Cursor-%EB%AC%B8-%EC%82%AC%EC%9A%A9%EB%B0%A9%EB%B2%95)
* [프로시저/예외처리](https://msdn.microsoft.com/ko-kr/library/ms175976.aspx)
