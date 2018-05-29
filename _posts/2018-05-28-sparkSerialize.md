---
layout: post
title: Spark serialize에대한 깨달음!
date: 2018-05-28 13:10
categories: Spark
---

# Java spark streaming을 scala로 바꾸면서 발생한 시행착오 공유

* spark job 처리 시 transform 작업이 간단한 경우 lambda 식으로 바로 적기도 하지만 재사용 등의 이유로 클래스로 만들어 사용하기도 한다
* 일반적인 경우에는 처리 프로세스들의 모음이므로 클래스의 인스턴스가 매번 새롭게 생성될 필요가 없으므로 object(java의 static)로 만들어서 사용한다
* 하지만 처리 프로세스 안에서 외부의 값을 받아서 써야 하는 일이 생겨 class로 정의하고 인스턴스를 만들어 전달을 하게 되었다 -> <span style="color:#e11d21">문제의 시작</span>

## 문제 발생 - 1

* 처리 프로세스들을 class로 묶으면서 hbase에 접근하여 값을 가져오는 처리가 필요했다
* hbase connection은 재사용이 가능하므로 한 번만 만들어 놓고 계속 쓰는 것이 효율적이라 클래스 변수로 hbase connection을 선언하고 클래스 생성 시 hbase connection에 필요한 값들얼 파라메터로 받아 connection을 생성했다

``` scala
class Functions(props: Properties) extends Serializable {
    @transient
    lazy val hbaseConnection = {
        val hbaseConf = HBaseConfiguration.create()
        props.entrySet().asScala.foreach(entry => {
        val key = entry.getKey.toString
        if (key.startsWith("hbase")) {
          hbaseConf.set(key, entry.getValue.toString)
        }
      })
      ConnectionFactory.createConnection(hbaseConf)
    }
    ...
    def getItem(log: Log) = {
        val table = hbaseConnection.getTable("itemdb")
        ...
    }
}

```

``` scala
Object Driver {
    ....
    val funtions = new Functions(props)
    ...
    streaming.map(functions.getItem)
}

```

* transform을 통해 executor로 instance(위에서는 funtions)가 전달되려면 serialize가 가능해야 하므로 Serializable을 implements 하고, hbaseConnection이 serializable하지 않으므로 transient 키워드를 붙여 serialize 처리 시 빠지게 하고 lazy 키워드를 붙여 처음 사용 시 한 번 생성하도록 했다
* 위와 같은 코드로 streaming을 실행하고 시간이 지나자 zookeeper connection과 관련한 문제가 발생하였다
    * hbaseConnection이 생성될때마다 zookeeper 연결도 같이 하게 되는데 hbaseConnection이 많아져서 zookeeper와 관련된 오류가 발생하는 것을 확인하였다
* <span style="color:#e11d21">driver를 통해 전달된 Funtions instance가 executor에 한 번 전달되면 deserialize된 hbaseConnection이 lazy가 적용되어 최초 한 번 호출 시 생성이 되고 그 다음부터는 재사용될 것이라고 생각했는데 streaming이 처리되는 주기마다 매 번 새로 생성이 되었다</span>
* transient와 lazy를 같이 써서 hbaseConnection이 매번 생성되는 것인가 생각을 하고 lazy를 빼고 적용을 하니 zookeeper 연결과 관련된 문제가 발생하지 않아 문제가 해결됐다고 생각했다 -> <span style="color:#e11d21">착각은 자유</span>
    * deserialize를 통해 executor에서 instance가 생성되고 transient된 필드는 코드대로 초기화가 수행될 것이라고 생각했는데 초기화부분이 처리되지 않아 hbaseConnection이 null이었고, 그로 인해 hbase에서 값을 가져오지 못하는 문제가 발생하였다

### Spark와 Serialize

* spark는 transform 처리 시 외부에서 생성된 instance를 serialize하여 executor로 보내고 executor에서는 deserialize하여 다시 instance를 만들어 이를 사용한다
* 해당 transform이 수행될 때마다 executor는 매 번 deserialize를 수행하며 이 때 매 번 새로운 instance가 생성된다
    * 일반적인 spark job의 경우 별 문제가 되지 않지만 streaming의 경우