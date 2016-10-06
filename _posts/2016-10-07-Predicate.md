---
layout: post
title: Java 8 Predicate
date: 2016-10-07 08:04
categories: Java
---

# Predicate

삼항연산자은 `X ? {true,false}`라고 적는다. 이를 Predicate로 표현하면 `preicate on X`라고 말해주면 된다. 

`java 8`에서는 Predicate 함수를 인터페이스로 만들어 lambda 표현식이나 method reference로 사용할 수 있게 하였다.

## Collection에서 Predicate사용


```java
package predicateExample;
 
public class Employee {
    
   public Employee(Integer id, Integer age, String gender, String fName, String lName){
       this.id = id;
       this.age = age;
       this.gender = gender;
       this.firstName = fName;
       this.lastName = lName;
   } 
     
   private Integer id;
   private Integer age;
   private String gender;
   private String firstName;
   private String lastName;
 
   //Please generate Getter and Setters
 
    @Override
    public String toString() {
        return this.id.toString()+" - "+this.age.toString(); //To change body of generated methods, choose Tools | Templates.
    }
}
```

직원 객체를 만들었다. 자 이제 `Predicate`를 사용해보자!!

### 1) 직원 중 남자이고 나이가 21살이상인 사람 찾기!


```java

public static Predicate<Employee> isAdultMale() {
    return p -> p.getAge() > 21 && p.getGender().equalsIgnoreCase("M");
}
```

### 2) 직원 중 여자이고 나이가 18살이상인 사람 찾기!


```java
public static Predicate<Employee> isAdultFemale() {
    return p -> p.getAge() > 18 && p.getGender().equalsIgnoreCase("F");
}
```


### 3) 직원 중 주어진 나이보다 많은 사람 찾기!!

```java
public static Predicate<Employee> isAgeMoreThan(Integer age) {
    return p -> p.getAge() > age;
}
```

세가지 Predicate를 하나의 클래스로 만들어 관리하자


```java
package predicateExample;
 
import java.util.List;
import java.util.function.Predicate;
import java.util.stream.Collectors;
 
public class EmployeePredicates 
{
    public static Predicate<Employee> isAdultMale() {
        return p -> p.getAge() > 21 && p.getGender().equalsIgnoreCase("M");
    }
     
    public static Predicate<Employee> isAdultFemale() {
        return p -> p.getAge() > 18 && p.getGender().equalsIgnoreCase("F");
    }
     
    public static Predicate<Employee> isAgeMoreThan(Integer age) {
        return p -> p.getAge() > age;
    }
     
    public static List<Employee> filterEmployees (List<Employee> employees, Predicate<Employee> predicate) {
        return employees.stream().filter( predicate ).collect(Collectors.<Employee>toList());
    }
}   
```

여기서 `filterEmployees()` method를 볼 수 있다. 간단하게 filter를 구현하고 코드가 if문을 남발하지 않아도 되는 예쁜코드를 작성할 수 있게 되었다.

이를 사용한 예를 보자!!

```java
package predicateExample;
 
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import static predicateExample.EmployeePredicates.*;
 
public class TestEmployeePredicates {
    public static void main(String[] args){
        Employee e1 = new Employee(1,23,"M","Rick","Beethovan");
        Employee e2 = new Employee(2,13,"F","Martina","Hengis");
        Employee e3 = new Employee(3,43,"M","Ricky","Martin");
        Employee e4 = new Employee(4,26,"M","Jon","Lowman");
        Employee e5 = new Employee(5,19,"F","Cristine","Maria");
        Employee e6 = new Employee(6,15,"M","David","Feezor");
        Employee e7 = new Employee(7,68,"F","Melissa","Roy");
        Employee e8 = new Employee(8,79,"M","Alex","Gussin");
        Employee e9 = new Employee(9,15,"F","Neetu","Singh");
        Employee e10 = new Employee(10,45,"M","Naveen","Jain");
         
        List<Employee> employees = new ArrayList<Employee>();
        employees.addAll(Arrays.asList(new Employee[]{e1,e2,e3,e4,e5,e6,e7,e8,e9,e10}));
                
        System.out.println(filterEmployees(employees, isAdultMale()));
         
        System.out.println(filterEmployees(employees, isAdultFemale()));
         
        System.out.println(filterEmployees(employees, isAgeMoreThan(35)));
         
        //Employees other than above collection of "isAgeMoreThan(35)" can be get using negate()
        System.out.println(filterEmployees(employees, isAgeMoreThan(35).negate()));
    }
}
 
Output:
 
[1 - 23, 3 - 43, 4 - 26, 8 - 79, 10 - 45]
[5 - 19, 7 - 68]
[3 - 43, 7 - 68, 8 - 79, 10 - 45]
[1 - 23, 2 - 13, 4 - 26, 5 - 19, 6 - 15, 9 - 15]
```

## java8에서 Predicates의 지향점

1. They move your conditions (sometimes business logic) to a central place. This helps in unit-testing them separately.

2. Any change need not be duplicated into multiple places. It improves manageability of code.

3. The code e.g. `filterEmployees(employees, isAdultFemale())` is very much readable than writing a if-else block.



<br/>

### Reference
* [HowToDoInJava](http://howtodoinjava.com/java-8/how-to-use-predicate-in-java-8/)
