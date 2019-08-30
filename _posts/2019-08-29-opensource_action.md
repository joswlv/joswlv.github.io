---
layout: post
title: OpenSource활동
date: 2019-08-29 15:10
categories: Think
---

# 올해 목표 조기달성!

2019년 1월 올해는 꼭 해야할 일을 정리했다. 그 중 `OpenSource활동하기`라는 막연한 계획이 있었다. 

2월에 Cassandra에 BulkLoad하는 기능이 필요해 이것 저것 찾던 중에 요구사항에 만족할 만한 것들이 없었다. 그래서 한번 내가 만들어 볼까라는 생각을 했고, 그렇게 회사 프로젝트에 CassandraBulkLoad 모듈을 만들어 적용했다. 하지만 원시 소스형태였고, 사용하기 불편한 형태의 코드였다.

작업한 코드는 BulkLoad하는 동안 Cassandra Read성능이 크게 떨어지지 않고, BulkLoad를 하는 방법이 단순해서 이거 Spark Lib으로 제작해보자라는 생각만 가지고 시간을 보냈다.

8월에 회사를 퇴사하게되면서, 자유시간이 많아졌다. 2월부터 생각만 했던 작업이 생각났고 자연스레 코드를 다시보았다. 그리고 Spark Lib형태로 제작하기 시작했다. 그렇게 탄생한게 [Spark2CassandraBulkLoad](https://github.com/joswlv/Spark2CassandraBulkLoad)이다.

해당 현재 해당 Repo는 Star가 5개이고, 대부분 프로젝트 내용을 알고 있는 회사사람들이다.

Lib을 제작하면서, 처음으로 [JCenter](https://bintray.com/joswlv/spark-utils/Spark2CassandraBulkLoad)에 배포도 해보고 [spark-package.org](https://spark-packages.org)에 등록도 하였다.

퇴사를 하면서 intellij License가 만료되어 걱정이였는데, 해당 repo로  JetBrains에 [opensource lincense](https://www.jetbrains.com/community/opensource/)를 신청해 1년치 All Products License 받았다.

3년 회사생활를 잘 마무리한 것 같아 뿌듯하다. 다음 주부터 새로운 집에서 새로운 회사로 출근한다. 

2019년 남은 4개월동안 새로운 곳에서 의미있는 일을 한번 해보자!


