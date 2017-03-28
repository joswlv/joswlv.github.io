---
layout: post
title: Maven Build Plugin
date: 2017-03-27 09:04
categories: Etc
---

# Maven Build Plugin


## 1. 디렉토리에 Resource 디렉토리 추가 방법

Maven 빌드를 할때, `src` 이외에 디록토리를 추가가 필요 할때가 있다.

방법은 다음과 같다.

```xml
<build>
<resources>
	<resource>
		<directory>bin</directory>
		<targetPath>${basedir}/target/bin</targetPath>
	</resource>
	<resource>
		<directory>conf</directory>
		<targetPath>${basedir}/target/conf</targetPath>
	</resource>
</resources>

...

</build>
```

`bin`, `conf` 디렉토리를 빌드 할때 함께 추가한다고 명시하면된다.


## 2. 빌드한 jar를 저장 위치를 변경하는 법

`src`디렉토리의 java파일을 컴파일해 만들어진 jar파일은 기본 /target아래에 만들어진다. 

이 위치를 변경하고 싶으면 다음과 같다.


```xml
<build>
	<plugins>
		<plugin>
			<groupId>org.apache.maven.plugins</groupId>
			<artifactId>maven-jar-plugin</artifactId>
			<configuration>
				<outputDirectory>${basedir}/target/lib</outputDirectory>
			</configuration>
		</plugin>
	</plugins>

...

</build>
```

## 3. dependency를 포함한 jar 만들기

이 방법은 여러가지가 있다. 그 중 assembly plugin을 사용하면 다음과 같다.

```xml
<plugin>
    <artifactId>maven-assembly-plugin</artifactId>
    <configuration>
        <descriptorRefs>
            <descriptorRef>jar-with-dependencies</descriptorRef>
        </descriptorRefs>
    </configuration>
    <executions>
        <execution>
            <id>make-assembly</id>
            <phase>package</phase>
            <goals>
                <goal>single</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

## 4. dependency jar를 모아 저장해 빌드하는 법

dependency에 있는 라이브러리 jar파일을 사용자가 지정한 디렉토리에 저장하는 방법은 다음과 같다.

```xml
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-dependency-plugin</artifactId>
	<executions>
		<execution>
			<id>copy-dependencies</id>
			<phase>package</phase>
			<goals>
				<goal>copy-dependencies</goal>
			</goals>
			<configuration>
				<outputDirectory>${basedir}/target/lib</outputDirectory>
				<overWriteReleases>false</overWriteReleases>
				<overWriteSnapshots>false</overWriteSnapshots>
				<overWriteIfNewer>true</overWriteIfNewer>
			</configuration>
		</execution>
	</executions>
</plugin>
```
