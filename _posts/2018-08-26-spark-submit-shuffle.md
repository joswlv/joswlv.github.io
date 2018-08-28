---
layout: post
title: Spark-submit shuffle서비스 이용시 주의점
date: 2018-08-26 15:10
categories: Spark
---

# Shuffle 서비스 이용시 주의할 점

```bash
spark-submit \
	--master yarn-cluster
	--conf spark.dynamicAllocation.enabled=true \
	--conf spark.shuffle.service.enabled=true
	...
	
```

`dynamicAllocation` 옵션을 사용할때 항상 같이 사용하는 옵션이 `spark.shuffle.service.enabled=true`이다. 하지만 해당 shuffle 옵션은 `yarn-cluster`모드에서는 동작하지 않는다.

이 문제를 해결하기 위해서는 `spark-submit`을 할때 shuffle서비스 관련 jar를 함께 배포하는 것이다.

아래와 같이 작성한다.

```bash
spark-submit \
	--master yarn-cluster
	--conf spark.dynamicAllocation.enabled=true \
	--conf spark.shuffle.service.enabled=true \
	--jars "/spark/lib/spark-1.6.3-yarn-shuffle.jar,/spark/lib/datanucleus-rdbms-3.2.9.jar,/spark/lib/datanucleus-core-3.2.10.jar,/spark/lib/datanucleus-api-jdo-3.2.6.jar" \
	...
```

spark 2.x대와 1.x대 버전에서 shuffle관련 jar path가 다르다. 2.x대 버전에서는 1.x대 버전에 있는 여러개 jar파일들을 하나로 합친것 같다.

* spark 1.6버전에서 shuffle관련 jar path ==> 
	-`$sparkHome/lib/spark-1.6.3-yarn-shuffle.jar`
	-`$sparkHome/lib/datanucleus-rdbms-3.2.9.jar`
	-`$sparkHome/lib/datanucleus-core-3.2.10.jar`
	-`$sparkHome/lib/datanucleus-api-jdo-3.2.6.jar`
	
* spark 2.x버전에서 suffle관련 jar path ==>
	-`$sparkHome/yarn/spark-2.1.1-yarn-shuffle.jar`
	