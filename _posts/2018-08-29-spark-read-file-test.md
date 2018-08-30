---
layout: post
title: Spark에서 Resource파일 읽을때
date: 2018-08-29 15:10
categories: Spark
---

# Spark에서 Resource파일 읽을때..

대부분의 spark job을 처리할때 hadoop에 처리할 파일을 업로드하여 사용한다. 하지만 Meta file이나 config file같이 용량이 얼마 되지 않은 파일을 Hadoop에 업로드하여 사용하기에는 부담이 있다.(HDFS 블록갯수가 많아지는 이슈)

그래서 resource(Maven프로젝트시)에 관련 파일을 읽는 법이 필요하다.

## 그럼 Scala(Java)에서 Resouce파일을 어떻게 읽었나?

총 4가지 방법이 존재 한다.

- classLoader에서 getResourceAsStream을 이용한 방식 (file이름은 상대경로)
- classLoader에서 getResource을 이용한 방식 (file이름은 상대경로)
- class에서 getResourceAsStream을 이용한 방식 (file이름은 절대경로)
- class에서 getResource을 이용한 방식 (file이름은 절대경로)

예제 소스는 다음과 같다.

<script src="https://gist.github.com/joswlv/591a9f039fe9d4dfb69d1afb97d45735.js"></script>

평소에 위와 같이 많이 사용한다. 이번에 Spark에서 Meta File을 읽는 과정에서 `getResource`를 사용했지만 동작이 되지 않아 테스트를 진행했다.

## 그럼 Spark에서는??

위 예제소스를 가지고 Test를 진행했다. 

```scala
def main(args: Array[String]): Unit = {
	val sc = new SparkContext()

	//class에서 getResource(AsStream)을 이용한 방식 (상대경로 절대경로 test도 추가)
	readFileByResourceAsStream("config.txt")
	readFileByResourceAsStream("/config.txt")
	readFileByResource("r1/config.txt")
	readFileByResource("/r1/config.txt")

	//classLoader에서 getResource(AsStream)을 이용한 방식 (상대경로 절대경로 test도 추가)
	readFileByClassLoaderResourceAsStream("r1/config.txt")
	readFileByClassLoaderResourceAsStream("/r1/config.txt")
	readFileByClassLoaderResource("r1/config.txt")
	readFileByClassLoaderResource("/r1/config.txt")

	println("class loaders start")
	var loader = SparkResourceTest.getClass.getClassLoader
	do {
		println(s"class loader : ${loader}")
	} while ( {
		loader = loader.getParent;
		loader != null
	})
	println("class loaders end")
}
```

결과는 다음과 같았다.

```
read file by ResourceAsStream : config.txt
getClass file read error : java.lang.NullPointerException


read file by ResourceAsStream : /config.txt
file line : 1,a
file line : 2,b
after read file


read file by Resource : config.txt
getClass file read error : java.lang.NullPointerException


read file by Resource : /config.txt
getClass file read error : java.nio.file.FileSystemNotFoundException


read file by ClassloaderResourceAsStream : config.txt
file line : 1,a
file line : 2,b
after read file


read file by ClassloaderResourceAsStream : /config.txt
getClass file read error : java.lang.NullPointerException


read file by ClassloaderResource : config.txt
getClass file read error : java.nio.file.FileSystemNotFoundException


read file by ClassloaderResource : /config.txt
getClass file read error : java.lang.NullPointerException
```

정리하면 

- classLoader에서 getResourceAsStream을 이용한 방식 (file이름은 상대경로) => 동작함!
- classLoader에서 getResourceAsStream을 이용한 방식 (file이름은 절대경로)
- classLoader에서 getResource을 이용한 방식 (file이름은 상대경로)
- classLoader에서 getResource을 이용한 방식 (file이름은 절대경로)
- class에서 getResourceAsStream을 이용한 방식 (file이름은 절대경로) => 동작함!
- class에서 getResourceAsStream을 이용한 방식 (file이름은 상대경로)
- class에서 getResource을 이용한 방식 (file이름은 절대경로)
- class에서 getResource을 이용한 방식 (file이름은 상대경로)

class, classLoader에 상관없이 getResourceAsStream만 동작하였다. 

왜 getResourceAsStream만 동작할까 Scala나 Java에서는 getResource도 사용가능한데...

Spark와 Scala(Java)에 차이가 있다면 File URL Path를 찾을 때 참조하는 classLoader 갯수가 달랐다.

#### spark

- org.apache.spark.util.MutableURLClassLoader@63648ee9
- sun.misc.Launcher$AppClassLoader@5c647e05
- sun.misc.Launcher$ExtClassLoader@5733f295

#### Scala(Java)

- sun.misc.Launcher$AppClassLoader@5c647e05
- sun.misc.Launcher$ExtClassLoader@5733f295

`MutableURLClassLoader`클래스가 private이라 디버깅은 못해봤지만,,,

Spark는 File URL Path를 찾아 나가는 방식이 Scala(Java)와는 반대 방향이고 찾는 규칙이 Scala(Java)보다 한단계 더 있을 것으로 추측된다.

## 결론

Spark에서 Resouce File을 읽을 때 절대경로로 읽고 싶으면 `class.getResourceAsStream` 상대경로로 읽고 싶으면 `class.getClassLoader.getResourceAsStream`으로 사용하면 된다.


