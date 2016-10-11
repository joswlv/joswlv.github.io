---
layout: post
title: <읽기좋은코드> 요약
date: 2016-10-11 21:04
categories: Etc
---

# 원본

* [\<읽기 좋은 코드가 좋은 코드다\>](http://www.hanbit.co.kr/book/look.html?isbn=978-89-7914-914-2)
* [NHN신입강의](https://nhnent.dooray.com/share/posts/1kprIA7VR3S4lwuMCa4D9Q)

# 코드는 이해하기 쉬워야한다.

## 간결한 코드가 좋다.

```java
for (Node* node = list->head; node != NULL; node = node->next)
    Print(node->data);
```
```java
Node* node = list->head;
if (node == NULL) return;

while(node->next != NULL) {
    Print(node->data);
    node = node->next;
}
if (node != NULL) Print(node->data);
```

## != 적은 줄

```java
return exponent>=0 ? mantissa = (1 << exponent) : mantissa / (1 <<-exponent);
```
```java
if (exponent >= 0) {
    return mantissa * (1 << exponent);
} else {
    return mantissa / (1 << -exponent);
}
```

**핵심**

`코드는 "다른 사람"이 그것을 "이해"하는 데 들이는 시간을 최소화하는 방식으로 작성해야한다.`

* "이해"
	* 코드를 자유롭게 수정하고
	* 버그를 짚어내고
	* 수정한 내용이 여러분이 작성한 다른 부분의 코드와 어떻게 상호작용하는지 알 수 있어야
* "다른 사람"
	* "지금의 나"와 "6개월 뒤의 나"는 다른 사람 


# 이름에 정보 담기!!

## 특정 단어 고르기

`GetPage()`

```java
def GetPage(url):
    ...
```

* get은 지나치게 보편적
* 로컬 캐쉬에서? 인터넷에서?!
	* `FetchPage()`, `DownloadPage()`

`Size()`

```java
class BinaryTree{
    int Size();
    ...
}
```

* 트리의 높이? Height()
* 노드 개수? NumNodes()
* 트리의 메모리? MemoryBytes()

`Stop()`

```java
class Thread {
    void Stop();
}
```

* 더 정확한 이름을 고를 수 있음
* 되돌릴 수 없는 최종 동작이라면 `Kill()`
* `Resume()`등으로 다시 돌이킬 수 있다면 `Pause()`

**보다 다양한 단어를 활용**

|단어|	유의어|
|----|--------|
|send	|deliver, dispatch, announce, distribute, route|
|find	|search, extract, locate, recover|
|start	|launch, create, begin, open|
|make	|create, set up, build, generate, compose, add, new|

# tmp, retval같은 보편적 이름 피하기

`retval`

* 리턴값이라고 무조건 `retval`을 쓰지 않기
* 그 변수가 의미하는 바를 이름으로

`tmp`

> tmp라는 이름은 대상이 짧게 임시적으로만 존재하고, 임시적 존재 자체가 변수의 가장 중요한 용도일 때 한해서 사용해야한다.


`i`,`j`,`k`

* 반복자라는 의미를 충분히 전달. 그래서 다른 목적으로 사용하면 헷갈릴 수 있음
* 다음과 같이 중첩해서 사용할 때 헷갈리지 않도록 유의. 조금 더 명확하게


```java
for (int i = 0; i < clubs.size(); i++)
  for (int j = 0; j < clubs[i].members.size(); j++)
      for (int k = 0; k < users.size(); k++)
          if (clubs[i].members[k] == users[j])
              ...
```
이런 경우는 

`i` --> `club_i`, `j` --> `members_j`, `k` --> `users_k`

이런 식으로 명확하게 한다.

> tmp, it, retval 같은 보편적인 이름을 사용하려면, 꼭 그렇게 해야하는 이유가 있어야 한다.


# 추상적인 이름보다 구체적인 이름을 선호하라

# 드모르간의 법칙 사용

```java
if (!(file_exists && !is_protected)) Error("...");
```

```java
if(!file_exists || is_protected)) Error("...");
```

> 영리하게 작성한 코드에 유의하라. 나중에 다른 사람이 읽으면 그런 코드가 종종 혼란을 초래한다.


# 변수와 가독성 

* 변수의 수가 많을수록 기억하고 다루기 더 어려워진다.
* 변수의 범위가 넓어질수록 기억하고 다루는 시간이 더 길어진다
* 변수값이 자주 바뀔수록 현재값을 기억하고 다루기가 더 어려워진다.

> 변수가 적용되는 범위를 최대한 좁게 만들어라


```java
class LargeClass {
    string str_;

    void Method1() {
        str_ = ...;
        Method2();
    }

    void Method2() {
        // str_ 변수 사용하기
    }

    // str_을 사용하지 않는 다른 메소드...
};
```

```java
class LargeClass {
    void Method1() {
        string str = ...;
        Method2(str);
    }

    void Method2(string str) {
        // str 변수 사용하기
    }
};
```

# 라이브러리와 주변 도구에 대한 충분한 이해가 필요하다.

* Java 7,8에서 추가된 API에 대한 이해
* Spring 버전 업 정보 확인




