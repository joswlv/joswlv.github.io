---
layout: post
title: Scala로 Retry 참고코드
date: 2018-10-24 15:10
categories: Scala
---

# Scala Retry 참고코드

### scala로 Retry 구현

```scala
// Returning T, throwing the exception on failure
@annotation.tailrec
def retry[T](n: Int)(fn: => T): T = {
  util.Try { fn } match {
    case util.Success(x) => x
    case _ if n > 1 => retry(n - 1)(fn)
    case util.Failure(e) => throw e
  }
}

// Returning a Try[T] wrapper
@annotation.tailrec
def retry[T](n: Int)(fn: => T): util.Try[T] = {
  util.Try { fn } match {
    case x: util.Success[T] => x
    case _ if n > 1 => retry(n - 1)(fn)
    case fn => fn
  }
}
```

### 예제

```scala 
val retryResult = retry(3) {
  if (validateObject(esClient)) {
    esClient
  } else {
    null
  }
}
```

### Reference

- [https://stackoverflow.com/questions/7930814/whats-the-scala-way-to-implement-a-retry-able-call-like-this-one](https://stackoverflow.com/questions/7930814/whats-the-scala-way-to-implement-a-retry-able-call-like-this-one)
