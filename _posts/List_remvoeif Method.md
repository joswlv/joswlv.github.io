---
layout: post
title: List.removeif Method
date: 2016-10-06 08:04
categories: Java
---

# List.remvoeif Method

`removeif()`메소드는 리스트내의 엘리먼트를 필터조건의 만족한는 것만을 골라 삭제할 때 사용된다.

**Package:** java.util

**Java Platform:** Java SE 8

**Syntax:**

```java
removeif(Prdicate<? super E> filter)
```

**형태**

```java
public boolean removeIf(Predicate<? super E> filter)
```

**예**

```java
import java.util.function.*;  
  
class SamplePredicate<t> implements Predicate<t>{  
  T varc1;  
  public boolean test(T varc){  
  if(varc1.equals(varc)){  
   return true;  
  }  
  return false;  
  }  
}  
```

```java
import java.util.*;  
  
public class test {  
  public static void main(String[] args) {  
   
  ArrayList<String> color_list;  
  SamplePredicate<String> filter;  
    
  color_list = new ArrayList<> ();  
  filter = new SamplePredicate<> ();  
    
  filter.varc1 = "White";  
    
 // use add() method to add values in the list  
    color_list.add("White");  
    color_list.add("Black");  
    color_list.add("Red");  
    color_list.add("White");  
    color_list.add("Yellow");  
    color_list.add("White");  
    
  System.out.println("List of Colors");  
  System.out.println(color_list);  
    
  // Remove all White colors from color_list  
  color_list.removeIf(filter);  
    
  System.out.println("Color list, after removing White colors :");  
  System.out.println(color_list);  
    
 }  
} 
```

**결과**

```
List of Colors
[White, Black, Red, White, Yellow, White]
Color list, after removing White colors :
[Black, Red, Yellow]
```

<br/>

### Reference
* [w3resource](http://www.w3resource.com/java-tutorial/arraylist/arraylist_removeif.php)
