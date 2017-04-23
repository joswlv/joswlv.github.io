---
layout: post
title: Resource CleanUp
date: 2017-04-20 09:04
categories: Java
---

# 리소스 클린업

java에서는 GC가 있어 리소스를 관리해준다. 하지만 외부 리소르를 클린업을 하지 않아 오작동을 발생하는 실수를 많이 하게 된다. 

단계별로 외부리소를 클린업을 하는 법을 정리하고, 최종적으로 람다표현식을 사용한 리소스클린업을 정리한다.

## 흔히하는 실수

```java
public class FileWriterEx {
    private final FileWriter writer;
    
    public FileWriterEx(final String fileName) throws IOException {
        writer = new FileWriter(fileName);
    }
    
    public void writeStuff(final String message) throws IOException {
        writer.write(message);
    }
    
    public void finalize() throws IOException {
        writer.close();
    }

    public static void main(String[] args) throws IOException {
        final FileWriterEx writerEx = new FileWriterEx("joswlvTest.txt");
        writerEx.writeStuff("TEST-TEST");
    }
}
```
이런 경우 `joswlvTest.txt`파일은 생성되지만 내용은 비어 잇다. `finalize()`가 실행되지 않았고, JVM은 충분한 메모리가 있기 때문에 `finalize()`를 할 필요가 없다고 생각한다. 그래서 파일은 종료되지 않고 작성한 내용은 메모리에서 파일로 flush가 되지 않았다.

## 수정 1

`흔히하는 실수` 문제를 수정하기 위해 명시적으로 `close()` 호출을 추가하고 `finalize()`메소드를 제거한다. 

인스턴스 사용이 끝나면, 해당 인스턴스를 클린업할 것을 명시적으로 요청할 수 있게 한것이다. 

```java
public class FileWriterEx {
    private final FileWriter writer;

    public FileWriterEx(final String fileName) throws IOException {
        writer = new FileWriter(fileName);
    }

    public void writeStuff(final String message) throws IOException {
        writer.write(message);
    }

    public void close() throws IOException {
        writer.close();
    }
    
    public static void main(String[] args) throws IOException {
        final FileWriterEx writerEx = new FileWriterEx("joswlvTest.txt");
        writerEx.writeStuff("TEST-TEST");
        writerEx.close();
    }
}
```

`close()`를 명시적으로 호출하는 것은 인스턴스가 사용한 외부 리소스를 클린업한다. 그러나 코드에서 예외처리가 일어나면 `close()` 메소드가 호출하지 못할 수도 있다. 

## 수정 2

`close()`메소드 호출을 보자하는 수정을 해보자

```java
public class FileWriterEx {
    private final FileWriter writer;

    public FileWriterEx(final String fileName) throws IOException {
        writer = new FileWriter(fileName);
    }

    public void writeStuff(final String message) throws IOException {
        writer.write(message);
    }

    public void close() throws IOException {
        writer.close();
    }

    public static void main(String[] args) throws IOException {
        final FileWriterEx writerEx = new FileWriterEx("joswlvTest.txt");
        try {
            writerEx.writeStuff("TEST-TEST");
        } finally {
            writerEx.close();
        }
    }
}
```

이렇게 하면 코드에서 예외가 발생하더라도 리소스에 대한 클린업이 보장된다.

## 수정 3

java SE7에서 부터 제공하는 `ARM (Automatic Resource Management)` 기능을 사용해 장황한 코드를 감소 시켜 보자!

```java
public class FileWriterEx implements AutoCloseable{
    private final FileWriter writer;

    public FileWriterEx(final String fileName) throws IOException {
        writer = new FileWriter(fileName);
    }

    public void writeStuff(final String message) throws IOException {
        writer.write(message);
    }

    public void close() throws IOException {
	    System.out.println("Close호출됨!");
        writer.close();
    }

    public static void main(String[] args) throws IOException {
        try (FileWriterEx writer = new FileWriterEx("joswlv.txt")) {
            writer.writeStuff("TEST-TEST");
        }
    }
}
```

클래스의 인스턴스를 `try-with-resources` 형태로 생성하고 그블록 안에서 writeStuff()를 호출한다. try 블록을 빠져나오면, 자동적으로 try 블록에 의해 관리되는 인스턴스/리소스에서 `close()` 메소드가 호출된다.

이러한 작업을 위해 컴파일러는 `AutoCloseable` 인터페이스의 구현에 대한 관리 리소스 클래스가 필요하고, 이 인터페이스는 `close()`라는 메소드 하나만 가진다.

ARM을 사용한 코드는 간결하지만 프로그래머가 원하는 시간에 클린업을 할려면 추가적인 작업이 필요하다.

## 수정 4

클래스가 일정 시간 내에 클린업이 되야하는 heavy한 리소스를 사용한다면 다음과 같이 하면 된다.


우선, 생성자와 `close()`를 `private`로 선언해 프로그래머가 직접 FileWriteEAM의 인스턴스를 생성할 수 없게 한다. 그래서 factory 메소드가 필요하다. 인스턴스를 생성하고 파라미터로 해당 인스턴스를 전달하는 일번적인 factory 메소드와 달리, 작성할 코드는 인스턴스를 사용자에게 전달하고 작업이 끝날 때까지 기다린다.(람다표현식을 이용하면 가능함)

```java

/*
FileWriterEAM.java
*/
public class FileWriterEAM {
    private final FileWriter writer;

    private FileWriterEAM(final String fileName) throws IOException {
        writer = new FileWriter(fileName);
    }

    public void writeStuff(final String message) throws IOException {
        writer.write(message);
    }

    private void close() throws Exception {
        System.out.println("Close 호출됨");
        writer.close();
    }

    public static void use(final String fileName, 
    	final UseInstance<FileWriterEAM, IOException> block) throws Exception 
    {
        final FileWriterEAM writerEAM = new FileWriterEAM(fileName);
        try {
            block.accept(writerEAM);
        } finally {
            writerEAM.close();
        }
        
        /* ARM으로 작성해도 된다. (AutoCloseable implements를 받아야한다.)
        try (FileWriterEAM writerEAM = new FileWriterEAM(fileName)) {
            block.accept(writerEAM);
        }
        */
    }

    public static void main(String[] args) throws Exception {
        FileWriterEAM.use("eam.txt", writerEAM -> writerEAM.writeStuff("GOOD!!"));
    }
}

/*
UseInstance.java
*/
@FunctionalInterface
public interface UseInstance<T, X extends Throwable> {
    void accept(T instance) throws X;
}
```

`use()` 메소드에서 두개의 파라미터 fileName, UseInstacne 인터페이스에 대한 레퍼런스를 인수로 받는다. 이 메소드에서 FileWriteEAM을 객체화하고 try와 finally 블록을 생성한다. 그리고 나서 인터페이스가 생성되자마자 `accept()` 메소드에 FileWriteEAM의 인스턴스를 파라미터로 넘긴다. 호출이 리턴되면, finally 블록에 잇는 인스턴스의 `close()`메소드를 호출한다. 


`@FunctionalInterface` Annotation를 붙여 UseInstance는 함수형 인터페이스로 만들어 자바 컴파일러가 자동으로 람다 표현식이나 메소드 레퍼런스를 합성할 수 있도록 한다.

클레스의 사용자들은 인스턴스를 직접 생성하지 못한다. 이런 특징은 인스턴스가 만료되는 시점 이후로 리소스의 클린업일어나는 코드 생성을 막는 효과를 가져온다. 


### Reference 

* 자바 8 람다의힘