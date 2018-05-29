---
layout: post
title: Scala 공변성에 대해서
date: 2018-05-27 13:10
categories: Scala
---

# Covariance에 대해서

Covariance(공변성)에 대해서 서술하기전에 Polymorphism(다형성)에 대해서 알아보자

### Polymorphism

- 어떤 함수가 여러 타입의 인자에 적용가능한지.
- 어떤 타입이 여러 타입의 인스턴스를 가질 수 있는지.
- Subtyping : Subclass의 인스턴스는 Superclass로 전달 가능
- Generics : 함수나 클래스를 Type Parameter를 통해 생성 가능

매개변수 다형성은 정적 타입의 장점을 포기하지 않으면서 일반적 코드(즉 여러 타입의 값을 처리할 수 있는 코드)를 만들기 위해 사용한다.

### Type Parameters

```scala
trait List[T] {
	def isEmpty: Boolean
	def: head: T
	def tail: List[T]
}
class Cons[T](val head: T, val tail: List[T]) extends List[T] {
	def isEmpty = false
}
class Nil[T] extends List[T] {
	def isEmpty = true
}

def singleton[T](elem: T) = 
	new Cons[T](elem, new Nil[T]
	
singleton[Int](1)
singleton[Boolean](true)
```

java의 generic같은 느낌이다.

### Upper Type Bounds

```scala
abstract class Animal {
	def name: String
}

abstract class Pet extends Animal {}

class Cat extends Pet {
	override def name = "Cat"
}

class Dog extends Pet {
	override def name = "Dog"
}

class Lion extends Aniaml {
	override def name = "Lion"
}

class PetContainer[P](p:P) {
	def pet: P = p
}
	
val dogContainer = new PetContainer(new Dog)
val catContainer = new PetContainer(new Cat)
val lionContainer = new PetContainer(new Lion). // <- 주목
```

PetContainer에 Type은 무엇이든 올 수 있다.
이때 Type을 한정할 수 있는데, 이를 Type Bounds라고 한다. 

- S <: T S는 T의 Subtype임
- S >: T T는 S으 Subtype임

예를 들어 PetContainer의 타입을 Pet이하의 것으로 한정하면 다음과 같이 된다.

```scala
class PetContainer[P <: Pet](p: P) {
	def pet: P = p
}

val dogContainer = new PetContainer(new Dog)
val catContainer = new PetContainer(new Cat)
val lionContainer = new PetContainer(new Lion). // <- 컴파일 Error
```

Pet은 Animal을 상속 받기 때문에 Lion이 PetContainer에 들어 갈 수 없다.

### Covariance(공변성)

앞선 예제를 생각해보자.
어떤 객체가 `Dog <: Pet`이면 `List[Dog] <: List[Pet]`일까?

List가 공변성을 가지고 있으면 맞는 말이다. 
만약 List가 반 공변적이면 Error이다.

scala의 List는 공변적이다.

```scala
abstract class List[+A]
```

다시 정리하면 하위타입 관계가 실제 의미하는 것은 “어떤 타입 A에 대해 B이 하위 타입(A <: B)이라면, B으로 A를 대치할 수 있는가?” 하는 문제이다(역주: 이를 리스코프치환원칙(Liskov Substitution Principle이라 한다.) 

이는 상위타입이 쓰이는 곳에는 언제나 하위 타입의 인스턴스를 넣어도 이상이 없이 동작해야 한다는 의미이다. 이를 위해 하위 타입은 상위 타입의 인터페이스(자바의 인터페이스가 아니라 외부와의 접속을 위해 노출시키는 인터페이스)를 모두 지원하고, 상위타입에서 가지고 있는 가정을 어겨서는 안된다).

```scala
C[T], A <: B

C[A] <: C[B]		convariant - 공변적
C[A] >: C[B]		contravariant - 반공변적
C[A]    C[B]		invariant - 무공변적

class C[+A] {...}	convariant - 공변적
class C[-A] {...}	contravariant - 반공변적
class C[A] {...}		invariant - 무공변적
```

예제를 보면

```scala
def process(animals: List[Animal]) = ...

val cats = List(new Cat())
val dogs = List(new Dog())

process(cats)
process(dogs)
```

List가 공변적이기 때문에 Aniaml의 하위 type을 List에 담을 수 있다.
만약 List가 반공변적이면 Animal보다 상위 type만 List에 담을 수 있다.

그래서 다음 예제도 어디가 틀렸는지 바로 알 수 있다.

```scala
val dogs = Array(new Dog)
val pets: Array[Pet] = dogs   <--Error
pets(0) = new Cat
val dog = dogs(0)
```

