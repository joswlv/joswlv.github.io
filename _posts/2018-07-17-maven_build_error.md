---
layout: post
title: Maven Build fail경험
date: 2018-07-17 09:04
categories: Etc
---

# Maven build시 Received fatal alert: protocol_version 오류 해결법

## 발생한 문제점

```xml
<dependency>
    <groupId>it.nerdammer.bigdata</groupId>
    <artifactId>spark-hbase-connector_2.10</artifactId>
    <version>1.0.3</version>
</dependency>
```

위 dependency를 이용하는 프로젝트를 jekins을 이용하여 maven build를 할려고 했다. 

하지만 다음과 같은 문제가 발생하여 빌드가 진행되지 않았다.

![]({{ site.url }}/images/maven_build_error.png)

여기서 주목할 점은 `Received fatal alert: protocol_version`이다!! 

maven repo에서 다운로드를 못 하고 있는 것이다.

문제를 찾아보니 `TLS library` 문제라고 한다. client에서 사용하는 java버전에 따라서 TLS이 기본으로 활성화 되있거나 비활성화 되있다고 한다.
불행하게도 회사 jekins의 기본 java version은 java7이고 [공식 문서](https://docs.oracle.com/javase/7/docs/technotes/guides/security/SunProviders.html#SunJSSEProvider)에 따르면 java7에서는 TLS버전 1.2을 지원하지만 default상태는 비활성화 되어 있다.

## 해결 방안

해결 방안은 두가지 방법이 존재하는데, 

1. Mavne build시 TLS사용을 강제로 지정해주는 것이다. 방법은 다음과 같다.
    - Maven  build 옵션에 `-Dhttps.protocols=TLSv1.2`을 추가해준다.
2. Java버전을 최신 버전으로 유지한다. (최소 java `1.8.0_60-b27`보다 높은 버전을 사용해야한다.)

두가지 방법중 어느 것을 택해도 문제는 해결된다.

## 참고

- [https://github.com/technomancy/leiningen/issues/2364](https://github.com/technomancy/leiningen/issues/2364)
- [https://github.com/jenkinsci/ghprb-plugin/issues/638](https://github.com/jenkinsci/ghprb-plugin/issues/638)