---
layout: post
title: Short-circuit evaluation
date: 2016-11-18 15:04
categories: Java
---

# Short-circuit evaluation이란

> `and`, `or`논리 연산에서 인자 하나만 보고 결과를 확실히 알 수 있을 때, 뒤에 ㄴ오는 인자를 확인하지 않고 바로 답을 내는 방법이다. `&&` 연산일 때, 하나라도 false이면 무조건 답이 false이고 || 연산일 땐, 하나라도 true이면 무조건 답이 true이다. 지원하는 언어는 [여기](https://en.wikipedia.org/wiki/Short-circuit_evaluation#Support_in_common_programming_languages)

예를 보면 다음과 같다.

```
조건식 1 && 조건식 2
조건식 1 이 false 이면, 조건식 2 를 건너뜀
```

#### 설명

`&&`연산자에서는 두 조건식이 모두 true가 되어야 전체 결과가 true가 되는데, `조건식1`이 `false`가되면 `조건식2`을 보지 않아도 전체가 `false`가 되므로 `조건식2`를 체크하지 않는다.


# 적용한 예

```java
if (check1 == '' && check2 == null){
	blabla
}
```

위 경우에서 `''`을 먼저 체크하는 것보다 `null`인 경우를 먼저 체크하는 것이 안전하다고 한다.

수정하면 아래와 같다.

```java
if (check2 == null && check1 == ''){
	blabla
}
```

<br/>



### Reference

* [ohyecloudy pnotes](http://ohyecloudy.com/pnotes/archives/542/)
* 