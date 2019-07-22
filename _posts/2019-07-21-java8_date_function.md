---
layout: post
title: Java8 date관련 함수 정리
date: 2019-07-21 13:10
categories: Java
---

# Java8 Date함수 정리

### 제발쫌 기억하자!!

현재 그리고 앞으로의 프로젝트에서 java8이하 버전을 사용할 가능성이 거의 없다고 생각한다. 그래서 java8 `java.time.*`을 정리하고자 한다.

(데이터를 날짜별로 분류하거나 로드할때 날짜 연산이 많이 사용되는데, 할때마다 날짜관련 메소드가 햇갈려서 이번에 정리하고자 한다.)

## 클래스

### 1. 날짜 (Temporal)

- Instant : machine time에 유용한 1970년 1월 1일부터 시간을 세는 클래스 (millisecond 뿐만 아니라 nanosecond까지 센다)
- LocalDate : [년,월,일]과 같은 날짜만 표현하는 클래스 (시간은 포함하지 않는다)
- LocalDateTime : [년,월,일,시,분,초]를 표현하는 클래스
- LocalTime : [시,분,초]와 같이 시간만 표현하는 클래스

### 2. 기간 (TemporalAmount)

- Period : 두 날짜 사이의 [년,월,일]로 표현되는 기간 (시간을 다루지 않다 보니 LocalDate를 사용한다)

- Duration : 두 시간 사이의 [일,시,분,초]로 표현되는 기간 (Instant 클래스를 사용하고, seconds와 nanoseconds로 측정 되지만 [일,시,분,초]로 변환해 주는 메쏘드를 제공)

### 기타

- ChronoUnit : 한가지의 단위를 표현하기 위한 클래스 (년,월,일,시,분,초 등)
- DayOfWeek : 요일


## 기억하기!

### 날짜 가져오기

```java
  LocalDate.now(); // 오늘
  LocalDateTime.now(); // 지금
  LocalDate.of(2019, 7, 22); // 2019년7월22일
  LocalDateTime.of(2019, 7, 22, 23, 23, 50); // 2019년7월22일23시23분50초
  Year.of(2017).atMonth(7).atDay(22).atTime(10, 30); // 2019년7월22일 10시30분00초
```

### 기간 가져오기

```java
  Period.ofYears(2); // 2년간(P2Y)
  Period.ofMonths(5); // 5개월간(P5M)
  Period.ofWeeks(3); // 3주간(P21D)
  Period.ofDays(20); // 20일간(P20D)

  Duration.ofDays(2); // 48시간(PT48H)
  Duration.ofHours(8); // 8시간(PT8H)
  Duration.ofMinutes(10); // 10분간(PT10M)
  Duration.ofSeconds(30); // 30초간(PT30S)
```
  
### 날짜 + 기간 = 날짜

```java
  LocalTime.of(9, 0, 0).plus(Duration.ofMinutes(10)); // (9시 + 10분간) = 9시10분
  LocalDate.of(2019, 7, 22).plus(Period.ofDays(1)); // (2019년7월22일 + 1일간) = 2019년7월23일
  LocalDateTime.of(2019, 7, 1, 23, 47, 5).minus(Period.ofWeeks(3)); // (2019년7월1일 23시47분05초 - 3주간) = 2019년6월10일 23시47분05초
  LocalDate.now().plusDays(1); // (오늘 + 1일) = 내일
  LocalTime.now().minusHours(3); // (지금 - 3시간) = 3시간 전
```

### 날짜 - 날짜 = 기간

```java
  Period.between(LocalDate.of(1950, 6, 25), LocalDate.of(1953, 7, 27)); // (1953년7월27일 - 1950년6월25일) = 3년1개월2일간(P3Y1M2D)
  Period.between(LocalDate.of(1950, 6, 25), LocalDate.of(1953, 7, 27)).getDays(); // 3년1개월2일간 => 2일간
  LocalDate.of(1950, 6, 25).until(LocalDate.of(1953, 7, 27), ChronoUnit.DAYS); // 3년1개월2일간 => 1128일간
  ChronoUnit.DAYS.between(LocalDate.of(1950, 6, 25), LocalDate.of(1953, 7, 27)); // 3년1개월2일간 => 1128일간

  Duration.between(LocalTime.of(10, 50), LocalTime.of(19, 0)); // (19시00분00초 - 10시50분00초) = 8시간10분간(PT8H10M)
  Duration.between(LocalDateTime.of(2015, 1, 1, 0, 0), LocalDateTime.of(2016, 1, 1, 0, 0)).toDays(); // 365일간
  ChronoUnit.YEARS.between(LocalDate.of(2015, 5, 5), LocalDate.of(2017, 2, 1)); // 1년간
```
  
### 날짜 변환하기

#### - LocalDate -> String

```java
LocalDate.of(2020, 12, 12).format(DateTimeFormatter.BASIC_ISO_DATE); // 20201212
```

### - LocalDateTime -> String

```java
LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")); // 2015-04-18 00:42:24
```

### - LocalDateTime -> java.util.Date

```java
Date.from(LocalDateTime.now().atZone(ZoneId.systemDefault()).toInstant()); // Sat Apr 18 01:00:30 KST 2015
```

### - LocalDate -> java.sql.Date

```java
Date.valueOf(LocalDate.of(2015, 5, 5)); // 2015-05-05
```

### - LocalDateTime -> java.sql.Timestamp

```java
Timestamp.valueOf(LocalDateTime.now()); // 2015-04-18 01:06:55.323
```

### - String -> LocalDate

```java
LocalDate.parse("2002-05-09"); // 2002-05-09
LocalDate.parse("20081004", DateTimeFormatter.BASIC_ISO_DATE); // 2008-10-04
```

### - String -> LocalDateTime

```java
LocalDateTime.parse("2007-12-03T10:15:30"); // 2007-12-03T10:15:30
LocalDateTime.parse("2010-11-25 12:30:00", DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")); // 2010-11-25T12:30
```

### - java.util.Date -> LocalDateTime

```java
LocalDateTime.ofInstant(new Date().toInstant(), ZoneId.systemDefault()); // 2015-04-18T01:16:46.755
```

### - java.sql.Date -> LocalDate

```java
new Date(System.currentTimeMillis()).toLocalDate(); // 2015-04-18
```

### - java.sql.Timestamp -> LocalDateTime

```java
new Timestamp(System.currentTimeMillis()).toLocalDateTime(); // 2015-04-18T01:20:07.364
```

### - LocalDateTime -> LocalDate

```java
LocalDate.from(LocalDateTime.now()); // 2015-04-18
```

### - LocalDate -> LocalDateTime

```java
LocalDate.now().atTime(2, 30); // 2015-04-18T02:30
```


### - 요일로 날짜 가져오기

```java
  LocalDate.now().with(TemporalAdjusters.next(DayOfWeek.SATURDAY)); // 다음 토요일
  LocalDate.of(2016, 5, 1).with(TemporalAdjusters.dayOfWeekInMonth(3, DayOfWeek.SUNDAY)); // 2016년5월 세번째 일요일
  LocalDate.of(2015, 7, 1).with(TemporalAdjusters.firstInMonth(DayOfWeek.MONDAY)); // 2015년7월 첫번째 월요일
```
  
### - 언어별 출력

```java
  DayOfWeek.MONDAY.getDisplayName(TextStyle.FULL, Locale.ENGLISH); // Monday
  DayOfWeek.MONDAY.getDisplayName(TextStyle.NARROW, Locale.ENGLISH); // M
  DayOfWeek.MONDAY.getDisplayName(TextStyle.SHORT, Locale.ENGLISH); // Mon

  DayOfWeek.MONDAY.getDisplayName(TextStyle.FULL, Locale.KOREAN); // 월요일
  DayOfWeek.MONDAY.getDisplayName(TextStyle.NARROW, Locale.KOREAN); // 월
  DayOfWeek.MONDAY.getDisplayName(TextStyle.SHORT, Locale.KOREAN); // 월

  Month.FEBRUARY.getDisplayName(TextStyle.FULL, Locale.US); // February
  Month.FEBRUARY.getDisplayName(TextStyle.FULL, Locale.KOREA); // 2월
```


### Reference

- [https://jekalmin.tistory.com/entry/%EC%9E%90%EB%B0%94-18-%EB%82%A0%EC%A7%9C-%EC%A0%95%EB%A6%AC](https://jekalmin.tistory.com/entry/%EC%9E%90%EB%B0%94-18-%EB%82%A0%EC%A7%9C-%EC%A0%95%EB%A6%AC)
