---
layout: post
title: Java Default Method에 정리
date: 2018-10-25 15:10
categories: Java
---

# Java Default Method에 정리

## Default Method란?

공개된 API에 새로운 메서드를 추가하면 기존 구현에 새롭게 오버라이딩을 해야하는 문제가 발생하여 기존구현을 수정하지 않으면 문제가된다. 즉 다음 세개의 호환성을 만족 시키기 위해 Default Method가 존재한다. 

- 바이너리 호환성 (뭔가를 바꾼 이후에도 에러없이 기존 바이너리가 실행될 수 있는 상황)
- 소스 호환성 (코드를 고쳐도 기존 프로그램을 성공적으로 재컴파일할 수 있는 상황)
- 동작 호환성 ( 코드를 바꾼 다음에도 같은 입력값이 주어지면 프로그램이 같은 동작을 하는 상황)

Default Method 사용방식은 간단하다. 

```java
public interface Example {
	int ex();
	//메소드 앞에 default만 붙이면 된다.
	default boolean isEmpty() {
		return ex() == 0;
	}
}
```

Default Method를 사용하면 인터페이스에 새로운 기능을 추가해도 하위버전의 호환이 보장이 된다.

## Default Method의 상속문제

다음과 같은 상황은 어떻게 해석할까?

```java
public interface A {
	default void hello() {
		System.out.println("Hello from A");
	}
}

public interface B extends A{
	default void hello() {
		System.out.println("Hello from B");
	}
}

public class C implements B, A {
	public static void main(String args[]) {
		new C().hello();  //"Hello from B"가 출력됨!
	}
}
```

이런 상황은 잘 없겠지만 다음 세가지 규칙을 알고 있으면 해석이 가능하다.

1. 클래스가 항상 이긴다. 클래스나 슈퍼클래스에서 정의한 메서드가 디폴트 메서드보다 우선권을 갖는다.
2. 1번 규칙 이외의 상황에서는 서브인터페이스가 항상 이긴다. 상속관계를 갖는 인터페이스에서 같은 시그니처를 갖는 메서드를 정의할 때는 서브인터페이스가 이긴다. 즉 B가 A를 상속받는 다면 B가 A를 이긴다.
3. 여전히 디폴트 메서드의 우선순위가 결정되지 않았다면 여러 인터페이스를 상속받은 클래스가 명시적으로 디폴트 메서드를 오버라이드하고 호출해야한다.


몇 가지 예제를 살펴보자!

```java
public interface A {
	default void hello() {
		System.out.println("Hello from A");
	}
}

public interface B extends A{
	default void hello() {
		System.out.println("Hello from B");
	}
}

public class D implements A {
	//A hello를 재정의
	void hello() {
		System.out.println("Hello from D");
	}
}

public class C extends D implements B, A {
	public static void main(String args[]) {
		new C().hello();  //"Hello from D"가 출력됨!(1번규칙 때문에)
	}
}
```


```java
public interface A {
	default void hello() {
		System.out.println("Hello from A");
	}
}

public interface B {
	default void hello() {
		System.out.println("Hello from B");
	}
}

public class C implements B,A {}

/**
인터페이스 간에 상속관계가 없으므로 2번 규칙을 적용할 수 없다. 
그러므로 A와B의 hello메서드를 구별할 기준이 없다. 
그래서 "Error: class C inherits unrelated defaults for hello() from ypes B and A."라는 에러가 발생한다.
**/
```


그럼 충돌을 해결은 어떻게 할까?

명시적으로 어떤 인터페이스에서 사용할 지를 선택해준다.


```java 
public interface A {
	default void hello() {
		System.out.println("Hello from A");
	}
}

public interface B {
	default void hello() {
		System.out.println("Hello from B");
	}
}

public class C implements B,A {
	void hello() {
		B.super.hello(); //명시적으로 인터페이스 B의 메서드를 선택한다.
	}
}
```

마지막으로 다이아몬든 문제를 보자

```java 
public interface A {
	default void hello() {
		System.out.println("Hello from A");
	}
}

public interface B extends A { }

public interface C extends A { }

public class D implements B, C {
	public static void main(String args[]) {
		new D().hello(); //어떻게 출력될까?
	}
}
```

D는 B와 C중 누구의 디폴트 메서드 정의를 상속받았을까? 실제로 선택할 수 있는 메서드 선언은 하나뿐이다. 따라서 "Hello from A"가 출력된다.

만약 B에도 같은 시그니처의 디폴트메서드 hello가 있다면 어떻게 될까? 2번 규칙은 디폴트메소드를 제공하는 가장 하위의 인터페이스가 선택된다고 말한다. B는 A를 상속받으므로 B가 선택된다.

그리고 B와 C모드 같은 시그니처의 디폴트메서드 hello를 가지고 있다면 충돌이 발생하므로 에러를 출력한다.



	