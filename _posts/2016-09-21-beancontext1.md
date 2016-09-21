---
layout: post
title: 스프링 IoC컨테이너 ApplicationContext [1]
date: 2016-09-21 16:29
categories: Spring
---

# XML 기반 빈 관리 컨테이너

스프링에서는 자바 객체를 `빈(Bean)`이라고 한다. 그래서 객체관리 컨테이너를 `빈컨테이너`라고 한다.

(스프링 IoC컨테이너 == 빈컨테이너)

**ApplicationContext의 계층도**

> * ApplicationContext (interface)
>	* ClasspathXmlApplicationContext (class)
>	* FileSystemXmlApplicationContext (class)
>	* WebApplicationContext (interface)

스프링에서 빈 정보는 XML 파일에 저장해 두고 ClassPathXmlApplicationContext 클래스나 FileSystemXmlApplicaionContext 클래스를 사용하여 빈을 자동 생성한다. ClassPathXmlApplicationContext는 자바 클래스 경로에서 XML로 된 빈 설정 파일을 찾는다. FileSystemXmlApplicationContext는 파일 시스템 경로에서 빈 설정 파일을 찾는다. WebApplicationContext는 웹 어플리케이션을 위한 IoC 컨테이너로서 web.xml파일에 설정된 정보에 따라 XML파일을 찾는다.

<br/>

### Bean 설정 XML 파일준비

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
        
  <bean id="score" class="exam.test01.Score"/>
    
</beans>
```

bean태그는 `<beans xmlns="http://www.springframework.org/schema/beans"/>` 네임스페이스에 소속되어 있기 때문에 XML파서가 이해할 수 있다. 이는 자바 클래스에서 import하는 것과 비슷하다. 

위의 소스를 자바소스로 표현하면 아래와 같다.

```java
new exam.test01.Score();
```

 스프링 IoC 컨테이너가 생성할 자바 빈에 대한 정보는 `<bean>`태그로 선언한다. class속성에는 클래스 이름을 정한다. 주의 할점은 반드시 패키지 이름을 포함한 클래스 이름(fully qulified name)이어야 한다. id속성은 객체의 식별자이다. 빈컨테이너에서 객체를 꺼낼 때 이 식별자를 사용한다.
 
 
**스프링 IoC 컨테이너 사용**

```java
public class Test {

	public static void main(String[] args) {
		ClassPathXmlApplicationContext ctx = 
				new ClassPathXmlApplicationContext("exam/test01/beans.xml");
		
		Score score = (Score) ctx.getBean("score");
		
		System.out.println("합계:" + score.sum());
		System.out.println("평균:" + score.average());
	}
}
```

### 익명 Bean 선언

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
        
  <bean class="exam.test03.Score"/>
  <bean class="exam.test03.Score"/>
</beans>
```

```java
package exam.test03;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {

	public static void main(String[] args) {
		ClassPathXmlApplicationContext ctx = 
				new ClassPathXmlApplicationContext("exam/test03/beans.xml");
		
		System.out.println("[컨테이너에 보관된 객체의 이름 출력]");
		for (String name : ctx.getBeanDefinitionNames()) {
			System.out.println(name);
		}
		
		System.out.println("[exam.test03.Score#0의 별명 출력]");
		for (String alias : ctx.getAliases("exam.test03.Score#0")) {
			System.out.println(alias);
		}
		
		System.out.println("[익명 빈 꺼내기]");
		Score score1 = (Score) ctx.getBean("exam.test03.Score");
		Score score2 = (Score) ctx.getBean("exam.test03.Score#0");
		if (score1 == score2) System.out.println("score == score#0");
		
		Score score3 = (Score) ctx.getBean("exam.test03.Score#1");
		if (score1 != score3) System.out.println("score != score#1");
		
		System.out.println("[클래스 타입으로 빈 꺼내기]");
		Score score4 = (Score) ctx.getBean(exam.test03.Score.class);
	}
}

/*
[컨테이너에 보관된 객체의 이름 출력]
exam.test03.Score#0
exam.test03.Score#1

[exam.test03.Score#0의 별명 출력]
exam.test03.Score

[익명 빈 꺼내기]
score == score#0
score != score#1

[클래스 타입으로 빈 꺼내기]
Error
```

클래스 타입으로도 객체를 꺼낼 수 있다. 하지만 getBean()을 호출할 때 Score 클래스에 대한 정보를 담은 java.lang.Class객체를 넘겨서 빈컨테이너에서 이 클래스의 인스턴스를 찾는다. 이때 같은 타입의 객체가 여러 개 있을 경우 예외가 발생한다.

<br/>

### 생성자와 프로퍼티 설정

`<constructor-arg>` 엘러먼트를 이용하면 호출될 생성자를 지정할 수 있다.

생성자를 지정하는 방법은 아래와 같다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
        
  <bean id="score1" class="exam.test04.Score">
      <constructor-arg><value type="java.lang.String">홍길동</value></constructor-arg>
      <constructor-arg><value type="float">91</value></constructor-arg>
      <constructor-arg><value type="float">92</value></constructor-arg>
      <constructor-arg><value type="float">93</value></constructor-arg>
  </bean>
  //자바코드 -> new Score("임꺽정",91,91,91);
  
  <bean id="score2" class="exam.test04.Score">
      <constructor-arg><value>임꺽정</value></constructor-arg>
      <constructor-arg><value>81</value></constructor-arg>
      <constructor-arg><value>82</value></constructor-arg>
      <constructor-arg><value>83</value></constructor-arg>
  </bean> 
  //자바코드 -> new Score("임꺽정",81,82,83);
  
  <bean id="score3" class="exam.test04.Score">
      <constructor-arg type="java.lang.String" value="장보고"/>
      <constructor-arg type="float" value="71"/>
      <constructor-arg type="float" value="72"/>
      <constructor-arg type="float" value="73"/>
  </bean> 
  //자바코드 -> new Score("장보고",71,72,73);
  
  <bean id="score4" class="exam.test04.Score">
      <constructor-arg value="이순신"/>
      <constructor-arg value="100"/>
      <constructor-arg value="98"/>
      <constructor-arg value="99"/>
  </bean>
  //자바코드 -> new Score("이순신",100,98,99);
    
  <bean id="score5" class="exam.test04.Score">
      <constructor-arg value="70" index="3"/>
      <constructor-arg value="50" index="1"/>
      <constructor-arg value="강감찬" index="0"/>
      <constructor-arg value="60" index="2"/>
  </bean>
  //자바코드 -> new Score("강감찬",50,60,70;
</beans>
```

<br/>
프로퍼티값 설정은 다음과 같다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
        
  <bean id="score1" class="exam.test05.Score">
    <property name="name"><value>홍길동</value></property>
    <property name="kor"><value>100</value></property>
    <property name="eng"><value>95</value></property>
    <property name="math"><value>90</value></property>
  </bean>
  //자바코드 -> Score score1 = new Score();
  //			score1.setName('홍길동');
  //			score1.setKor(100);
  //			score1.setEng(59);
  //			score1.setMath(90);
  
  <bean id="score2" class="exam.test05.Score">
    <property name="name" value="임꺽정"/>
    <property name="kor" value="85"/>
    <property name="eng" value="99"/>
    <property name="math" value="100"/>
  </bean>
  //자바코드 -> Score score2 = new Score();
  //			score2.setName('임꺽정');
  //			score2.setKor(85);
  //			score2.setEng(99);
  //			score2.setMath(100);  
</beans>
```

<br/>

### `<bean>`의 속성을 이용하여 생성자 및 프로퍼티 설정하기

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:c="http://www.springframework.org/schema/c"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  
  <bean id="score1" class="exam.test06.Score"
      p:name="홍길동" p:kor="100" p:eng="95" p:math="90"/>
      
  //자바코드 -> Score score1 = new Score();
  //			score1.setName('홍길동);
  //			score1.setKor(100);
  //			score1.setEng(95);
  //			score1.setMath(90);
  
  <bean id="score2" class="exam.test06.Score"
      c:name="임꺽정" c:kor="80" c:eng="90" c:math="100"/>

  //자바코드 -> Score score2 = new Score('임꺽정',80,90,100);  
</beans>
```

`xmlns:p="http://www.springframework.org/schema/p"`, `xmlns:c="http://www.springframework.org/schema/c"`의 네임스페이스를 통해서 `<bean>`속성을 통해 프로퍼티와 생성자도 설정이 가능하다.

<br/>

### 의존 객체 주입

어떤 객체가 작업을 수행하기 위해 다른 객체를 지속적으로 사용한다면 그 사용되는 객체를 의존 객체(depencencies)라 한다. 보통 지속적으로 사용할 객체는 프로퍼티에 보관한다.

예제로 Car객체를 만들때 tires객체와 engine객체를 사용한다. 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:c="http://www.springframework.org/schema/c"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
  
  <bean id="engine1" class="exam.test07.Engine" 
      c:maker="Hyundai" p:cc="1998"/>
  //자바코드 -> Engine engine1 = new Engine("Hyundai");
  //			engine1.setCc(1998);			  
  
  <bean id="car1" class="exam.test07.Car">
      <property name="model"><value>Avante</value></property>
      <property name="engine"><ref bean="engine1"/></property>
  </bean>
  //자바코드 -> Car car1 = new Car();
  //			car1.setModel("Avante");
  //			car1.setEngine(engine1);  
  
  <bean id="car2" class="exam.test07.Car"
      c:model="Equus" c:engine-ref="engine1"/>
  //자바코드 -> Car car2 = new Car("Equus",engine1);
</beans>
```

`<ref>`태그를 사용하여 Engine의 참조하였다. 

<br/>

### 개별 인스턴스 주입하기

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
	xmlns:p="http://www.springframework.org/schema/p"
	xmlns:c="http://www.springframework.org/schema/c"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

	<bean id="car1" class="exam.test08.Car">
		<constructor-arg value="Avante" />
		<constructor-arg>
			<bean class="exam.test08.Engine" p:maker="Hyundai" p:cc="1495" />
		</constructor-arg>
	</bean>
		
	//자바코드 -> Engine temp = new Engine();
	//			  temp.setMaker("Hyundai");
	//			  temp.setCc(1495);
	//			  Car Car1 = new Car("Avante", temp);
	
	<bean id="car2" class="exam.test08.Car">
		<property name="model" value="Sonata" />
		<property name="engine">
			<bean class="exam.test08.Engine" p:maker="Hyundai" p:cc="1997" />
		</property>
	</bean>
	
	//자바코드 -> Car car2 new Car();
	//			  car2.setModel("Sonata");
	//			  Engine temp = new Engine();
	//			  temp.setMaker("Hyundai");
	//			  temp.setCc(1997);
	//			  car2.setEngine(temp);
</beans>
```

<br/>

### 컬랙션 값 주입

**배열 프로퍼티의 값 주입**


```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
	xmlns:p="http://www.springframework.org/schema/p"
	xmlns:c="http://www.springframework.org/schema/c"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

	<bean id="car1" class="exam.test09.Car">
		<constructor-arg value="Avante" />
		<constructor-arg>
			<bean class="exam.test09.Engine" p:maker="Hyundai" p:cc="1495" />
		</constructor-arg>
		<property name="tires">
			<list>
				<bean class="exam.test09.Tire" p:maker="Kumho" p:spec="P185" />
				<bean class="exam.test09.Tire" p:maker="Kumho" p:spec="P185" />
			</list>
		</property>
	</bean>
	
	//자바코드 -> Engine temp = new Engine();
	//			  temp.setMaker("Hyundai");
	//			  temp.setCc(1495);
	//			  Car car1 = new Car("Avante",temp);
	//			  Tire[] temp2 = new Tire[]{new Tire(), new Tire()};
	//			  temp2[0].setMaker("Kumho");
	//			  temp2[0].setSpeck("P185");
	//			  temp2[1].setMaker("Kumho");
	//			  temp2[1].setSpeck("P185");
	//			  car1.setTire(temp2);
</beans>
```
<br/>

**Map과 Properties값 주입**

>Map과 Preoperties의 용도
>
>java.util.Map 타입의 클래스는 식별자(key)나 값(value)으로 문자열뿐만 아니라 다른 타입의 객체도 사용할 수 있습니다. java.util.Properties 클래스도 Map의 일종(Map 인터페이스를 구현했음)이지만 주로 문자열로 된 식별자와 값을 다룰 때 사용한다.

<br/>


```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
	xmlns:p="http://www.springframework.org/schema/p"
	xmlns:c="http://www.springframework.org/schema/c"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

	<bean id="spareTire" class="exam.test10.Tire">
		<property name="maker" value="Hyundai" />
		<property name="spec">
			<props>
				<prop key="width">205</prop>
				<prop key="ratio">65</prop>
				<prop key="rim.diameter">14</prop>
			</props>
		</property>
	</bean>
	
	//자바 코드 -> Tire spareTire = new Tire();
	//				spareTire.setMaker("Hyundai");
	//				Properties temp = new Properties();
	//				temp.setProperty("width", "205");
	//				temp.setProperty("ratio","65");
	//				temp.setProperty("rim.diameter","14");
	//				spareTire.setSpec(temp);

	<bean id="car1" class="exam.test10.Car">
		<constructor-arg value="Avante" />
		<constructor-arg>
			<bean class="exam.test10.Engine" p:maker="Hyundai" p:cc="1495" />
		</constructor-arg>
		<property name="options">
			<map>
				<entry>
					<key>
						<value>sunroof</value>
					</key>
					<value>yes</value>
				</entry>
				<entry key="airbag" value="dual" />
				<entry key="sparetire">
					<ref bean="spareTire" />
				</entry>
			</map>
		</property>
	</bean>
	
	//자바 코드 -> Engine temp = new Engine();
	//				temp.setMaker("Hyundai")
	//				temp.setCc(1496);
	//				Car car1 = new Car("Avante",temp);
	//				Map<Object,Object> tempMap = new HashMap<Object,Object>();
	//				car1.setOptions(tempMap);
	//				tempMap.put("sunroof","yes");
	//				tempMap.put("airbag","dual");
	//				tempMap.put("spaertire",spareTire);
</beans>
```


<br/>

### Reference
* [열혈강의 자바 웹 개발 워크북](http://www.yes24.com/24/Goods/13159413?Acode=101)
