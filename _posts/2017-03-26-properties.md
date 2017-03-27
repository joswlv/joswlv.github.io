---
layout: post
title: properties 설정 및 사용
date: 2017-03-26 09:04
categories: Java
---

# Properties 사용하기


`properties`는 `Hashtable`를 상속받아 Key value로 구성 되어 있다.

key와 value를 `=`으로 구분한다.

다음은 properties 파일의 예이다.

```
spark.network.timeout=800
spark.driver.memory=1G
spark.executor.memory=1G
spark.rdd.compress=true
spark.storage.memoryFraction=1
spark.core.connection.ack.wait.timeout=600
spark.akka.frameSize=50
spark.streaming.backpressure.enabled=true
spark.serializer=org.apache.spark.serializer.KryoSerializer
```

이런식으로 `=`를 사용하여, 구분 짓는다. 

다음은 properties파일을 읽는 모듈이다.


```java
public static Properties getProperties(String configFilePath) {
        Properties prop = new Properties();
        InputStream in = null;
        try {
            prop.load(new FileInputStream(configFilePath));
        }catch (IOException e){
            System.out.println("Error >>>>> can't read the configFilePath => " + configFilePath);
            System.exit(-1);
        }finally {
            try {
                if (in != null)
                    in.close();
            }catch (IOException e) {}
            finally {
                return prop;
            }
        }
    }
```

```java
for (Map.Entry<Object, Object> ele : DefaultSetting.getProperties("resources/sites.txt").entrySet()) {
            System.out.println((String) ele.getKey() +" ==> "+ (String) ele.getValue());
        }
```

이런식으로 HashMap에 넣어서 사용하면 편리하다.

        