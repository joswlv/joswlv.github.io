---
layout: post
title: TRUNCATE와 DELETE
date: 2016-12-17 09:00
categories: MYSQL
---



# TRUNCATE 와 DELETE

### TRUNCATE

테이블의 데이터를 삭제하는 DDL명령어 이다.

데이터를 삭제 방식은 talbe의 명세만 남기고 **DROP 후 CREATE**한다. 즉 데이터가 존재하던 공간마저 제거하기 위한 명령어이다.

이런 방식으로 테이블의 데이터를 삭제하기 때문에 수행 속도는 DELETE보다 빠르지만, 데이터 복구가 불가능하다는 제한사항이 존재한다. 

또 auto_increment까지 초기화 시켜준다.(5.0.13버전부터) 즉 table의 옵션을 초기상태로 돌려준다.

삭제하다가 외부키와의 의존관계 때문에 삭제할 수 없다는 메시지가 나오면 다음과 같이하면 된다.

```
SET foreign_key_checks = 0; 
truncate TABLE_NAME;
SET foreign_key_checks = 1;
```

0은 해제 1은 다시 설정을 의미한다.

### DELETE 

테이블의 데이터를 삭제하기 위한 명령어이다. `WHERE`절을 통해 조건을 부여 할 수 있다. 

데이터 삭제 방식은 한줄씩 순차적으로 삭제한다. 
