---
layout: post
title: Checked Exception과 Uncecked Exception 정리
date: 2018-10-29 15:10
categories: Java
---

# Checked Exception과 Uncecked Exception 정리

## 1. 예외란? (Error vs Exception)

먼저 오류(Error)와 예외(Exception)의 개념을 정리하자.

**오류(Error)**는 시스템에 비정상적인 상황이 생겼을 때 발생한다. 이는 시스템 레벨에서 발생하기 때문에 심각한 수준의 오류이다. 따라서 개발자가 미리 에측하여 처리할 수 없기 때문에, 어플리케이션에서 오류에 대한 처리를 신경 쓰지 않아도 된다.

오류가 시스템 레벨에서 발생한다면, 예외(Exception)는 개발자가 구현한 로직에서 발생한다. 즉, 예외는 발생할 상황을 미리 예측하여 처리할 수 있다. 개발자가 처리할 수 있기 때문에 예외를 구분하고 그에 따른 처리방법을 명확히 알고 적용하는 것이 중요하다.

### Exception Class 계층도

![]({{ site.url }}/images/java-exception-handling-class-hierarchy-diagram.jpg)

## 2. Checked Eception과 Unchecked Exception

||Checked Exception | Unchecked Exception|
|-----|------|-------|
|처리여부|반드시 예외를 처리해야함|명시적인 처리를 강제하지 않음|
|확인 시점|컴파일단계|실행단계|
|예외발생시 트랜잭션처리| roll-back하지않음|roll-back함|
|대표 예외| Exeption의 상속받는 하위클래스 중 Runtime Exception을 제외한 모든 예외(대표적으로 IOException)|RuntimeException 하위 예외 (대표적으로 NullPointerException)|

좀더 자세히 보자!

#### Unchecked Exception

- Code를 잘못 만들어서 생기는 예외
- 컴파일하는데는 문제없다. 실행하면 문제가 발생함

배열의 범위를 벗어난다던가(IndexOutOfBoundsException)
값이 null인 참조변수의 멤버를 호출하려 했다던가(NullPointerException)
클래스간의 형변환을 잘못했다던가(ClassCastException)
정수를 0으로 나누려 했다던가(ArithmeticException)하는 경우에 발생하는 예외들이다. 

RuntimeException클래스들 중의 하나인 ArithmeticException을 try-catch문으로 처리하는 경우도 있지만, 
사실 try-catch문을 사용하기보다는 0으로 나누지 않도록 프로그램을 변경하는 것이 올바른 처리방법이다. 
이처럼 RuntimeException예외들이 발생할 가능성이 있는 코드들은 try-catch문을 사용하기 보다는 프로그래머들이 보다 주의 깊게 작성하여 예외가 발생하지 않도록 해야 할 것이다. 

그 외의 Exception클래스들은 주로 외부의 영향으로 발생할 수 있는 것들로서, 프로그램의 사용자들의 동작에 의해서 발생하는 경우가 많다. 예를 들면, 존재하지 않는 파일을 처리하려한다던지(FileNotFoundException), 실수로 클래스의 이름을 잘못 적었다던가(ClassNotFoundException), 입력한 데이터의 형식이 잘못되었다던가(DataFormatException) 하는 경우에 발생하는 예외들이다. 
이런 종류의 예외들은 반드시 처리를 해주어야 한다.

#### Checked Exception

- Exception 처리코드를 compiler가 check
- 프로그램 실행 흐름상 예외발생 가능성 있는 상황을 표현
- Code상의 문제가 아니라, 실행상황에 따라 발생가능성 있는 예외
- 프로그램 구현 흐름상 발생할 수 있는 예외

## 3. 예외 처리

- `try catch finally`	  - 직접처리
	- 다음메소드에 던져줄 때, 명시해줘야한다.
			      	 

- throws	       - 간접처리 	
	- 이걸 명시해주지않으면, 예외가 있는지 없는지 모르니까 명시해 준다.


- throw		        - 예외생성	 
	- Checked Exception에서 던져주는것 상위 메소드에다가 예외를 만들어서.



### 권장사항

- 왠만하면 Catch구문 안에는 Logic를 넣지 않는다. (보통 log만찍는다.)
- Catch구문은 많이 쓸수 있다.
(권장)Catch계층구조가 높을수록 아래로 내려야 한다.  - 자식이 처리할수 있는 건 자식이 하고 자식이 처리하지 못하는 것은 아래에 둔다.(부모가 처리)
- 순서가 바뀌면 UnReachableException이 발생한다.

