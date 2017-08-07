---
layout: post
title: sbt 기본 사용법
date: 2017-08-06 10:04
categories: Scala
---

# SBT 기본 사용법

### 1. SBT 프로젝트 디렉토리 구조 및 기본적인 명령어

```
|-src/
|--main/
|----resources/
|       <files to include in main jar here>
|----scala/
|       <main Scala sources>
|----java/
|       <main Java sources>
|--test/
|----resources
|       <files to include in test jar here>
|----scala/
|       <test Scala sources>
|----java/
       <test Java sources>
       
빌드 definition 파일 - 베이스 디렉토리에 있는 build.sbt 파일 
빌드 support 파일 - 베이스 디렉토리 하위 project라는 디렉토리에 포함한 파일들을 의미한다. 
빌드 target 디렉토리 - 기본으로 베이스 디렉토리 하위의 target
       
```

maven과 유사하다.

간단한 테스크를 실행하면 아래와 같다.

```bash
➜  scala-Learning-spark sbt clean package
[info] Loading global plugins from /Users/nhnent/.sbt/0.13/plugins
[info] Loading project definition from /Users/nhnent/dev/scala-Learning-spark/project
[info] Set current project to scala-Learning-spark (in build file:/Users/nhnent/dev/scala-Learning-spark/)
[success] Total time: 0 s, completed 2017. 8. 7 오후 8:14:27
[info] Updating {file:/Users/nhnent/dev/scala-Learning-spark/}scala-learning-spark...
[info] Resolving jline#jline;2.12.1 ...
[info] Done updating.
[info] Packaging /Users/nhnent/dev/scala-Learning-spark/target/scala-2.11/scala-learning-spark_2.11-1.0.jar ...
[info] Done packaging.
[success] Total time: 1 s, completed 2017. 8. 7 오후 8:14:28
```

**기본적인  명령어**

```
clean - 타겟 디렉토리에 생성된 모든 파일을 삭제한다.
compile - 메인 리소스에 있는 모든 소스를 컴파일 한다.
test - 컴파일을 하고 모든 테스트케이스를 수행한다.
package - src/main 하위의 자바와 스칼라의 컴파일된 클래스와 리소스를 패키징한다./java.
reload  - 빌드 설정을 리로드한다.
```

기본적인 사용법은 maven과 매우 유사한 것을 알 수 있다.

빌드 버전은 `project/build.properties`에서 설정할 수 있다.

### 2. Dependencies 추가하기

maven의 `pom.xml`파일처럼 sbt에서는 build.sbt에 명세한다.

intellij에서 sbt프로젝트를 생성하면 다음과 같이 적혀있다.

```
name := "scala-Learning-spark"

version := "1.0"

scalaVersion := "2.11.8"
```

규칙은 다음과 같다.

- 좌측부분은 키이다.
- 오퍼레이터 는 `:=`이다.
- 우측부분은 바디라고 불린다.

maven처럼 dependencies를 설정할려면 다음과 같이 추가 해주면 된다.

```
libraryDependencies ++= Seq(
  groupID % artifactID % revision,
  groupID % otherID % otherRevision
)
```

예를 들면 다음과 같다.

```
libraryDependencies ++= Seq(
  "org.apache.spark" %% "spark-core" % "1.6.0" % "provided",
  "org.apache.spark" %% "spark-sql" % "1.6.0" % "provided",
  "org.apache.spark" %% "spark-hive" % "1.6.0" % "provided"
)
```

기본적으로 Dependency 소스저장소는 `Maven2 repository`를 사용한다. 하지만 사내망안에 있는 reopsitory가 필요할 때가 있다. 이럴 때 maven에서는 `<repositories>`태그를 추가해서 해결 했지만 sbt는 다음과 같이 한다.

```
resolvers += name at location

#ex> resolvers += "Sonatype OSS Snapshots" at "https://oss.sonatype.org/content/repositories/snapshots"
```

### 3. assembly plugin사용법

maven에서 dependency libraries를 함께 packaging을 할 수 있었다. sbt에서도 똑같이 할 수 있는데 방법은 다음과 같다.

build.sbt에 다음 내용을 추가한면된다.

```
#jar이름 설정
assemblyJarName in assembly :=  artifact.value.name + "-" + version.value + "." + artifact.value.extension

###예시###
assemblyMergeStrategy in assembly := {
  case PathList("javax", "servlet", xs @ _*)         => MergeStrategy.first
  case PathList(ps @ _*) if ps.last endsWith ".html" => MergeStrategy.first
  case "application.conf"                            => MergeStrategy.concat
  case "unwanted.txt"                                => MergeStrategy.discard
  case x =>
    val oldStrategy = (assemblyMergeStrategy in assembly).value
    oldStrategy(x)
}
###예시 end###

assemblyMergeStrategy in assembly := {
  case PathList("com","payco",xs @ _*) => MergeStrategy.last
  case _ => MergeStrategy.discard
}
```

이렇게 설정한뒤에 다음 명령어를 사용하면 원하는 데로 packaging을 할 수 있다.

```
sbt clean assembly
```

### 4. 기본적인 오퍼레이션 설명

- **`:=`**
you can assign a value to a setting and a computation to a task. For a setting, the value will be computed once at project load time. For a task, the computation will be re-run each time the task is executed.

- **`+=`**
will append a single element to the sequence.

- **`++=`**
will concatenate another sequence.

예를들어 기본적으로 컴파일 폴더는 src/main/scala가 되는데, Compile의 Seq[File] 값을 가지는 sourceDirectories라는 키를 통해 컴파일 단계에서 source라는 디렉토리를 컴파일에 포함시키고 싶다면 아래와 같이 하면된다.

```scala
sourceDirectories in Compile += new File("source")
```

아니면  sbt 패키지에 포함되어 있는 file() 함수로 더 쉽게 할 수도 있다.

```scala
sourceDirectories in Compile += file("source")
```

(file() just creates a new File.)

++= 메소드를 사용하면 여러 디렉토리를 한번에 추가 하는 것도 가능하다.

```scala
sourceDirectories in Compile ++= Seq(file("sources1"), file("sources2"))
```

default source directory를 완전 수정하고 싶다면 `:=` 메소드를 사용하자.

```scala
sourceDirectories in Compile := Seq(file("sources1"), file("sources2")
```
