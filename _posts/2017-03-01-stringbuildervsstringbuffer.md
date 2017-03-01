---
layout: post
title: StringBuilder vs StringBuffer
date: 2017-03-01 09:04
categories: Java
---

# StringBuilder vs StringBuffer

### 단일 쓰레드환경에서 속도 비교

```java
/**
 * Buffers : 9
 * Builder : 4
 * strings:21756
 *
 * 단일 쓰레드 환경에서는 StringBuilder가 가장빠르다.
 */
public class StringSpeedTest {
    public static void main(String[] args) {


        long t0 = System.currentTimeMillis();

        t0 = System.currentTimeMillis();
        StringBuffer buf = new StringBuffer();
        for (int i = 0 ; i < 100000; i++){
            buf.append("some string");
        }
        System.out.println("Buffers : "+(System.currentTimeMillis() - t0));

        t0 = System.currentTimeMillis();
        StringBuilder building = new StringBuilder();
        for (int i = 0 ; i < 100000; i++){
            building.append("some string");
        }
        System.out.println("Builder : "+(System.currentTimeMillis() - t0));

        String withString ="";
        for (int i = 0 ; i < 100000; i++){
            withString+="some string";
        }
        System.out.println("strings:" + (System.currentTimeMillis() - t0));
    }
}
```

StringBuilder가 가장 좋은 성능을 보여줬다.

### 멀티쓰레드환경에서 비교

```java
/**
 * Created by Jo_seungwan on 2017. 3. 1..
 * StringBuilder는 java.lang.ArrayIndexOutOfBoundsException을 출력한다.
 */
public class StringMultiThreadTest {
    public static void main(String[] args) {

        ThreadPoolExecutor executorService = (ThreadPoolExecutor) Executors.newFixedThreadPool(10);
        //With Buffer
        StringBuffer buffer = new StringBuffer();
        for (int i = 0 ; i < 10; i++){
            executorService.execute(new AppendableRunnable(buffer));
        }
        shutdownAndAwaitTermination(executorService);
        System.out.println(" Thread Buffer : "+ AppendableRunnable.time);

        //With Builder
        AppendableRunnable.time = 0;
        executorService = (ThreadPoolExecutor) Executors.newFixedThreadPool(10);
        StringBuilder builder = new StringBuilder();
        for (int i = 0 ; i < 10; i++){
            executorService.execute(new AppendableRunnable(builder));
        }
        shutdownAndAwaitTermination(executorService);
        System.out.println(" Thread Builder: "+ AppendableRunnable.time);

    }

    static void shutdownAndAwaitTermination(ExecutorService pool) {
        pool.shutdown(); // code reduced from Official Javadoc for Executors
        try {
            if (!pool.awaitTermination(60, TimeUnit.SECONDS)) {
                pool.shutdownNow();
                if (!pool.awaitTermination(60, TimeUnit.SECONDS))
                    System.err.println("Pool did not terminate");
            }
        } catch (Exception e) {}
    }
}

class AppendableRunnable<T extends Appendable> implements Runnable {

    static long time = 0;
    T appendable;
    public AppendableRunnable(T appendable){
        this.appendable = appendable;
    }

    @Override
    public void run(){
        long t0 = System.currentTimeMillis();
        for (int j = 0 ; j < 10000 ; j++){
            try {
                appendable.append("some string");
            } catch (IOException e) {}
        }
        time+=(System.currentTimeMillis() - t0);
    }
}
```

멀티 쓰레드환경에서 10000개를 append하는 작업에서 `StringBuffer`속도는 느렸다.

그 이유는 JIT / hotspot / compiler / 무언가가 잠금을 검사 할 필요가 없다는 것을 탐지 할 때 최적화를 수행하기 때문이다.

이에 반해 `StringBuilder`는 java.lang.ArrayIndexOutOfBoundsException을 출력한다. 


### 정리

|구분 |StringBuffer|StringBuilder|
|----|----|-----|
|synchronized|Yes|No|
|Thread-safe|Yes|No|
|performance|Slow|Better than StringBuffer|



