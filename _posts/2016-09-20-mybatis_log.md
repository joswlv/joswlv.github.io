---
layout: post
title: mybatis log출력
date: 2016-09-20 11:29
categories: web
---

# 1. mybatis 로그 출력

데이터베이스 입출력을 로그로 볼 수 있는 기능이 mybatis에 있다. mybatis설정 파일에 다음과 같이 편집을 한다.



```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <settings>
    <setting name="logImpl" value="LOG4J"/>
  </settings>
  
  ...
  
```

`<setting>`태그를 사용하여 로그 출력기를 지정한다. 이 태그의 `name`속성은 로그 출력기를 호출할 이름이고,  `value`로그 출력기의 종류를 설정한다.

로그 출력기 종류는 다음과 같다.

`SLF4J`, `LOG4J`, `LOG4J2`, `JDK_LOGGING`, `COMMONS_LOGGING`, `STDOUT_LOGGING`, `NO_LOGGING`


다음 과정으로는 로그출력기의 설정파일을 만든다. 이 파일에는 로그의 수준, 출력방식, 출력형식, 로그 대상 등에 대한 정보가 들어 간다. 그리고 이 파일의 경로는 자바 클래스 경로(CLASSPATH)에 두어야 한다.

```
# Global logging configuration
log4j.rootLogger=ERROR, stdout

# MyBatis logging configuration...
log4j.logger.spms.dao=TRACE

# Console output...
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%5p [%t] - %m%n
```

<br/>

#### 로그의 출력 등급 설정


제일 먼저 루트 로거의 출력 등급을 ERROR로 설정했다. 루트 로거란 최상위 로거를 가리킨다. 하위 로거들은 항상 부모의 출력 등급을 상속 받는다. 하위 로거들의 등급을 별도로 설정하지 않으면 루트 로그의 출력 등급과 같은 ERROR가 된다.


**로그 출력 등급표**


| 로그 출력 등급 | 설명 |
|------------|-------|
| FATAL | 어플리케이션을 중지해야 할 심각할 오류 |
| ERROR | 오류가 발생 했지만, 어플리케이션은 계속 실행할 수 있는 상태 |
| WARN | 잠재적 위험을 안고 있는 상태 |
| INFO | 어플리케이션의 주요 실행 정보 |
| DEBUG | 어플리케이션의 내부 실행 상황을 추적해 볼 수 있는 상세 정보 |
|  TRACE | 디버그보다 더 상세한 정보 |

<br/>

#### 출력 담당자 선언 및 유형결정

루트 로거의 출력 등급을 설정하는 값 다음에는 출력 담당자(Appender)의 이름이 온다. 위의 소스에서는 출력 담당자의 이름을 `stdout`이라고 했다.

출력 담당자가 여럿일 때 당담자의 이름도 그 수만큼 지정한다. 이름은 마음데로 지정해도 된다.


출력 담당자 유형을 설정하는 문법은 다음과 같이 설정한다.

```
log4j.appender.이름=출력 담당자(페키지명을 포함한 클래스명)
```

<br/>


**주요 로그 출력자 클래스**


| 출력담당자클래스 | 설명 |
|---------|----------|
| org.apache.log4j.ConsoleAppender | System.out 또는 System.err로 로그를 출력한다. 기본은 System.out이다. 즉 표준 출력 장치인 모니터로 출력한다. |
| org.apache.log4j.FileAppender | 파일로 로그를 출력한다. |
| org.apache.log4j.net.Socketappender | 원격의 로그 서버에 로그 정보를 담은 LoggingEvent객체를 보낸다. |

<br/>


#### 로그의 출력 형식 정의

로그 출력 담당자를 결정했으면, 다음으로 로그의 출력 형식을 정의해야 한다. 문법은 다음과 같다.

```
log4j.appender.이름.layout=출력형식 클래스(패키지명을 포함한 클래스명)
```

<br/>

**출력 형식 클래스**

| 출력 형식 클래스 | 설명 |
|------------|------|
| org.apapche.log4j.SimpleLayout | 출력형식은 `출력등록-메시지`이다. <br/>출력 결과 예) <br/>DEBUG - DB Connection Error! |
| org.apache.log4j.HTMLLayout | HTML 테이블 형식으로 출력한다. |
| org.apache.log4j.PatternLayout | 변환 패턴의 형식에 따라 로그를 출력한다.  <br/>반환 패턴 예) <br/>%-5p[%t]:%m%n <br/>  출력 결과 예)<br/> DEBUG[main]:Message 1 <br/>WARN[main]:Message 2 |
| org.apache.log4j.xml.XMLLayout | log4j.dtd 규칙에 따라 XML을 만들어 출력한다.  <br/>출력 결과 예)<br/> \<log4j:eventSet version="1.2"xmlns:log4j="http://jakarta.apache.org/log4j/"\> &data: \</log4j:eventSet> |



<br/>

### Reference
* [열혈강의 자바 웹 개발 워크북](http://www.yes24.com/24/Goods/13159413?Acode=101)
