---
layout: post
title: TRUNCATE와 DELETE
date: 2017-01-05 10:00
categories: Java
---

# Java String

우선 Java의 `String` 객체는 immutable(불변의) 클래스이다. 그래서 한번 생성되면 수정이 불가능 하다.

다음은 `String`객체의 특성을 나열 한다.

### 1. "" or 객체 String 만들기

String을 만들때는 두가지 방식이 존재한다. 

```java
String a = "abc";
String b = new String("abc");
```

우선 차이를 설명하는 예를 보자

```java
String a = "abc";
String b = "abc";
System.out.println(a == b); //True
System.out.println(a.equals(b)); //True
```

`a`와 `b`는 Method Area에서 같은 메모리를 참조 한다.

```java
String c = new String("abc");
String d = new String("abc");
System.out.println(c == d); //False
System.out.println(c.equals(d)); //True
```
`c`와 `d`는 다른 heap영역에서 다른 object이다. 다른 object는 다른 메모리 참조한다.

String을 만들때 객체를 새로 만드는 것보다 `""`으로 String을 만드는 것이 불필요한 객체를 안만들어서 더 유용하다고 한다.

### 2. String은 passed by value

다음 예를 보자

```java
public static void main(String[] args) {
	String a = new String("abc");
	changeText(a);
	System.out.println(a);
}

public static void changeText(String a) {
	a = "cd";
}
```

이 예제에서 print되는 것은 "abc"이다.

이러한 이유는 Java는 항상 `pass by value`를 하기 때문이다. 즉 `changeText()`메소드를 지나면서 `a`에 저장되었던 "abc"는 무시되고 "cd"가 새롭게 heap생성이된다. 처음에 생성되었던 a는 여전히 "abc"를 참조하고 있다. 이는 앞서 설명한 Java에서는 String이 immutable한 클래스이기 때문이다. 즉 참조에 의한 값전달(passed by reference)는 지원하지 않는다.

immutable하지 않는 String을 만들고 싶으면 `StringBuilder`를 사용하면 된다.

다음 예는 `changeText()`메소드의 기능을 살리는 방식이다.

```java
public static void main(String[] args) {
	StringBuilder x = new StringBuilder("ab");
	changeText(x);
	System.out.println(x);
}
 
public static void changeText(StringBuilder x) {
	x.delete(0, 2).append("cd");
}
```
여기서 StringBuilder는 값이 변할 수 있다. 그리고 새로운 객체를 생성하는 것이 아니라 새로운 값을 할당한다.

요약하면 Java는 값(변수?)의한 전달만 가능하다!! 참조를 통한 전달은 지원하지 않는다.

C++로 `passed by reference`의 예를 보면 다음과 같다.

```c++
void change(string &x) {
    x = "cd";
}
 
int main(){
    string x = "ab";
    change(x);
    cout << x << endl;
}
```
최종적인 x는 "cd"가 출력된다. heap에 생성된 x의 메모리주소를 참조하여 값을 변경하였다.


### Reference

* [programcreek](http://www.programcreek.com/)
* [JavaDude.com](http://javadude.com/articles/passbyvalue.htm)
* [어제보다 나은 오늘](http://fowler.egloos.com/1243657)