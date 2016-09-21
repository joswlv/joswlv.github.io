---
layout: post
title: 스프링 IoC컨테이너 ApplicationContext [2]
date: 2016-09-21 20:29
categories: Spring
---

# XML 기반 빈 관리 컨테이너

### 팩토리 메서드와 팩토리 빈

객체 지향 설계 기법 중에서 팩토리 메서드(factory method) 패턴과 빌더(Builder)패턴이 있다. 

인스턴스를 생성하는 일반적인 방법은 new 명령을 사용하는 것인데, new 명령을 통해 클래스와 생성자를 지정하면 해당 클래스의 인스턴스가 생성되고 초기화 된다. 문제는 인스턴스 생성작업이 복잡한 경우이다. 매번 인스턴스를 생성할 때마다 복잡한 과정을 거쳐야 한다면 코딩할 때 부담이 된다. 이를 해결하기 위해 나온 것이 팩토리 메서드 패던과 빌더 패턴이다.

팩토리 클래스는 인스턴스 생성과정을 캡슐화 한다. 즉 인스턴스를 생성하고 초기화하는 복잡한 과정을 감추기 때문에 개발자입장에서는 인스턴스를 얻는 과정이 간결해진다. 

팩토리 메서드를 만들 때는 두가지 방법이 있다. 스태틱으로 선언하여 클래스 메서드로 만들거나 아니면 인스턴스 메서드로 만드는 것이다. 

먼저 스태틱으로 선언된 팩토리 메서드를 설정하는 방법을 보자

<br/>

```java
package exam.test11;

import java.text.SimpleDateFormat;
import java.util.Properties;

public class TireFactory {
	public static Tire createTire(String maker) {
		if (maker.equals("Hankook")) {
			return createHankookTire();
		} else {
			return createKumhoTire();
		}
	}
	
	private static Tire createHankookTire() {
		Tire tire = new Tire();
		tire.setMaker("Hankook");
		
		Properties specProp = new Properties();
		specProp.setProperty("width", "205");
		specProp.setProperty("ratio", "65");
		specProp.setProperty("rim.diameter", "14");
		tire.setSpec(specProp);
		
		SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
		try {
			tire.setCreatedDate(dateFormat.parse("2014-5-5"));
		} catch (Exception e) {}
		
		return tire;
	}
	
	private static Tire createKumhoTire() {
		Tire tire = new Tire();
		tire.setMaker("Kumho");
		
		Properties specProp = new Properties();
		specProp.setProperty("width", "185");
		specProp.setProperty("ratio", "75");
		specProp.setProperty("rim.diameter", "16");
		tire.setSpec(specProp);
		
		SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
		try {
			tire.setCreatedDate(dateFormat.parse("2014-3-1"));
		} catch (Exception e) {}
		
		return tire;
	}
}
```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
	xmlns:p="http://www.springframework.org/schema/p"
	xmlns:c="http://www.springframework.org/schema/c"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

	<bean id="hankookTire" class="exam.test11.TireFactory"
		factory-method="createTire">
		<constructor-arg value="Hankook" />
	</bean>
	//자바코드 -> Tire hankookTire = TireFactory.createTire("Hankook");
	
	<bean id="kumhoTire" class="exam.test11.TireFactory"
		factory-method="createTire">
		<constructor-arg value="Kumho" />
	</bean>
	//자바코드 -> Tire kumhoTire = TireFactory.createTire("kumho");
	
</beans>
```

여기서 중요한 것은 `factory-method`속성의 값이다. 이 값은 Tire 인스턴스를 생성할 메서드 이름이다. `factory-method`속성에는 반드시 static 메서드 이름을 지정해야 한다. 팩토리 메서드에 넘겨 줄 매개변수 값은 `<constructor-arg>`태그로 설정한다.


>**자바객체(Java object) vs 인스턴스(instance) vs 빈(bean)
>
>new 명령어나 팩토리 메서드로 생성된 `인스턴스`를 일반적으로 `자바 객체`라 부른다. 스프링에서는 자바 객체를 `빈`이라고 부른다. 일상 생활에서도 하나의 사물을 다양하게 표현하듯이, 예를 드렁 물을 음료라고 부리기도 하고 음료수, 마실것 등으로 부르는 것처럼, 프로그래밍에서도 하나의 개념을 다양한 말로 표현한다.


<br/>

다음으로 인스턴스 팩토리 메서드를 정의하고, 사용하는 예이다.

```java
package exam.test12;

import java.text.SimpleDateFormat;
import java.util.Properties;

public class TireFactory {
	public Tire createTire(String maker) {
		if (maker.equals("Hankook")) {
			return createHankookTire();
		} else {
			return createKumhoTire();
		}
	}
	
	private Tire createHankookTire() {
		Tire tire = new Tire();
		tire.setMaker("Hankook");
		
		Properties specProp = new Properties();
		specProp.setProperty("width", "205");
		specProp.setProperty("ratio", "65");
		specProp.setProperty("rim.diameter", "14");
		tire.setSpec(specProp);
		
		SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
		try {
			tire.setCreatedDate(dateFormat.parse("2014-5-5"));
		} catch (Exception e) {}
		
		return tire;
	}
	
	private Tire createKumhoTire() {
		Tire tire = new Tire();
		tire.setMaker("Kumho");
		
		Properties specProp = new Properties();
		specProp.setProperty("width", "185");
		specProp.setProperty("ratio", "75");
		specProp.setProperty("rim.diameter", "16");
		tire.setSpec(specProp);
		
		SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
		try {
			tire.setCreatedDate(dateFormat.parse("2014-3-1"));
		} catch (Exception e) {}
		
		return tire;
	}
}
```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
	xmlns:p="http://www.springframework.org/schema/p"
	xmlns:c="http://www.springframework.org/schema/c"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

  <bean id="tireFactory" class="exam.test12.TireFactory"/>
  //자바코드 -> TireFactory tireFactory = new TireFactory
  
	<bean id="hankookTire" factory-bean="tireFactory" factory-method="createTire">
		<constructor-arg value="Hankook" />
	</bean>
	//자바코드 -> Tire hankookTire = tireFactory.createTire("Hankook");

	<bean id="kumhoTire" factory-bean="tireFactory" factory-method="createTire">
		<constructor-arg value="Kumho" />
	</bean>
	//자바코드 -> Tire hankookTire = tireFactory.createTire("Kumho");
</beans>
```

`factory-bean`속성에 팩토리 객체(tireFactory)를 지정한다. class 속성은 설정하지 않는다. `factory-method`속성에는 인스턴스 팩토리 메서드 이름을 지정한다. 스태틱 메서드는 안된다.!! 팩토리 메서드의 매개변수 값은 이전과 마찬가지로 `<constructor-arg>`태그로 지정한다.

<br/>

### 스프링 규칙에 따라서 팩토리 빈 만들기

스프링에서는 팩토리 빈이 갖추어야 할 규칙을 `org.springframework.beans.factoryFactoryBean`인터페이스에 지정했다. 팩토리 클래스를 만들 때 이 규칙에 따라 메서드를 구현하면 된다.

>FactoryBean인터페이스 구현
>
>* [Interface] FactoryBean 
>	* getObject()
>	* getObjectType()
>	* isSingleton()

보통 FactoryBean 인터페이스를 직접 구현하기보다는 스프링에서 제공하는 추상클래스를 상속해서 만든다. `org.springframework.beans.factory.config.AbstractFactoryBean`은 FactoryBean인터페이스를 미리 구현했다.

>* [interface] FactoryBean 
>	* [abstract] AbstractFactoryBean - createInstance(), getObjectType()
>	* [class] TireFactoryBean

AbstractFactoryBean에는 createInstance()라는 추상 메서드가 잇다. 빈을 생성할 때 팩토리메서드로서 getObject()가 호출 되는데, getObject()는 내부적으로 바로 이 메서드를 호출한다. 즉 실제 빈을 생성하는 것은 createInstacne() 이다. 따라서 AbstractFactoryBean 클래스를 상속받을 때는 반드시 이 메서드를 구현해야 하고, 이 메서드에 빈 생성 코드를 넣는다. 

그리고 FactoryBean 인터페이스로부터 받은 getObjectType() 추상 메서드가 구현되지 않은 채로 남아 있다. 이 메서드는 팩토리 메서드(getObject())가 생성하는 객체의 타입을 알려 준다. 이 메서드는 반드시 구현해야한다.

```java
package exam.test13;

import java.text.SimpleDateFormat;
import java.util.Properties;

import org.springframework.beans.factory.config.AbstractFactoryBean;

public class TireFactoryBean extends AbstractFactoryBean<Tire> {
	String maker;
	
	public void setMaker(String maker) {
	  this.maker = maker;
  }
	
	@Override
	public Class<?> getObjectType() {
	  return exam.test13.Tire.class;
	}
	
	protected Tire createInstance() {
		if (maker.equals("Hankook")) {
			return createHankookTire();
		} else {
			return createKumhoTire();
		}
	}
	
	private Tire createHankookTire() {
		Tire tire = new Tire();
		tire.setMaker("Hankook");
		
		Properties specProp = new Properties();
		specProp.setProperty("width", "205");
		specProp.setProperty("ratio", "65");
		specProp.setProperty("rim.diameter", "14");
		tire.setSpec(specProp);
		
		SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
		try {
			tire.setCreatedDate(dateFormat.parse("2014-5-5"));
		} catch (Exception e) {}
		
		return tire;
	}
	
	private Tire createKumhoTire() {
		Tire tire = new Tire();
		tire.setMaker("Kumho");
		
		Properties specProp = new Properties();
		specProp.setProperty("width", "185");
		specProp.setProperty("ratio", "75");
		specProp.setProperty("rim.diameter", "16");
		tire.setSpec(specProp);
		
		SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
		try {
			tire.setCreatedDate(dateFormat.parse("2014-3-1"));
		} catch (Exception e) {}
		
		return tire;
	}
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
	xmlns:p="http://www.springframework.org/schema/p"
	xmlns:c="http://www.springframework.org/schema/c"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

	<bean id="hankookTire" class="exam.test13.TireFactoryBean">
		<property name="maker" value="Hankook" />
	</bean>
	//자바코드 -> FactoryBean tempFactoryBean = new TireFactoryBean();
	//			  tempFactoryBean.setMaker("Hankook");
	//			  Tire tireFactory = tempFactoryBean.getObject();
</beans>
```

<br/>

### 빈의 범위 설정

스프링 IoC컨테이너는 빈을 생성할 때 기본으로 한개만 생성한다. getBean()을 호출하면 계속 동일한 객체를 반환한다. 그러나 설정을 통해 이런 빈의 생성 방식을 조정할 수 있다. 다음은 스프링에서 설정 가능한 범위를 정리한 표이다. <br/>

| 범위 | 설명 |
|------|---------|
| singleton | 오직 하나의 빈만 생성한다.(기본설정) |
| prototype | getBean()을 호출할 때마다 빈을 생성한다. |
| request | HTTP요청이 발생할 때마다 생성되며, 웹 어플리케이션에서만 이 범위를 설정할 수 있다. |
| session | HTTP세션이 생성될 때마다 빈이 생성되며, 웹 어플리케이션에서만 이 범위를 설정할 수 있다. |
| globalsession | 전역 세션이 준비될 때 빈이 생성된다. 웹 어플리케이션에서만 이 범위를 지정할 수 있으며, 보통 포틀릿 컨텍스트에서 사용한다. |


**싱글톤과 프로토타입**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
	xmlns:p="http://www.springframework.org/schema/p"
	xmlns:c="http://www.springframework.org/schema/c"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

	<bean id="hyundaiEngine" class="exam.test14.Engine" 
	    p:maker="Hyundai" p:cc="1997"/>
	
	<bean id="kiaEngine" class="exam.test14.Engine"
	    p:maker="Kia" p:cc="3000"
	    scope="prototype"/>
</beans>
```

hyundaiEngine은 범위를 설정하지 않아서 기본 범위인 `singletone`으로 설정되었다.

kiaEngine은 scope속성의 값을 `prototype`으로 설정했다. 이 경우 빈 컨테이너에게 kiaEngine을 요청할 때마다 매번 새 Engine인스턴스를 만들어 반환할 것이다.

```java
package exam.test14;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {

	public static void main(String[] args) {
		ClassPathXmlApplicationContext ctx = 
				new ClassPathXmlApplicationContext("exam/test14/beans.xml");
		
		System.out.println("[singleton 방식(기본)]-------------------------");
		
		Engine e1 = (Engine) ctx.getBean("hyundaiEngine");
		Engine e2 = (Engine) ctx.getBean("hyundaiEngine");
		
		System.out.println("e1-->" + e1.toString());
		System.out.println("e2-->" + e2.toString());
		if (e1 == e2) {
			System.out.println("e1 == e2");
		}
		
		System.out.println("[prototype 방식]-------------------------");
		
		Engine e3 = (Engine) ctx.getBean("kiaEngine");
		Engine e4 = (Engine) ctx.getBean("kiaEngine");
		
		System.out.println("e3-->" + e3.toString());
		System.out.println("e4-->" + e4.toString());
		if (e3 != e4) {
			System.out.println("e3 != e4");
		}
	}
}

/*
[singleton 방식(기본)]
e1-->[Engine:Hyundai,1997]
e2-->[Engine:Hyundai,1997]
e1 == e2

[prototype 방식]
e3-->[Engine:Kia,3000]
e4-->[Engine:Kia,3000]
e3 != e4
*/
```

kiaEngine의 범위는 프로토타입(prototype)으로 설정되었다. 따라서 getBean()을 호출할 때마다 새로운 빈을 만들어서 반환한다. 그래서 e3와 e4가 다르다고 출력된다.


<br/>

### 애노테이션을 이용한 의존객체 자동 주입

스프링에서는 자바 애노테이션을 이용해 간단히 의존 객체를 주입할 방법을 제공해 준다. 빈의 셋터 메서드에 `@Autowired`를 선언하면 빈 컨테이너가 셋터의 매개변수 타입과 일치하는 빈을 찾아 자동으로 설정해 준다. 이 기능을 이용하려면 빈 설정 파일에 다음 객체를 선언해야 한다.

```xml
<bean class="org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor" />
```

`AutowiredAnnotationBeanPostProcessor`클래스는 빈의 후 처리기(post processor)로서 빈을 생성한 후 즉시 `@Autowired`로 선언된 셋터를 찾아서 호출하는 역할을 수행한다. 즉 `@Autowired`가 붙은 프로퍼티에 대해 의존 객체(프로퍼티 타입과 일치하는 빈)를 찾아 주입(프로퍼티 할당)하는 일을 한다.
<br/>

**`@Autowired`적용**

```java
public class Car {
	String							model; // 모델명
	Engine 							engine; // 엔진
	Tire[]							tires; // 타이어
	Map<String,Object> 	options; //선택사항
	
	public Car() {}
	
	public Car(String model, Engine engine) {
		this.model = model;
		this.engine = engine;
	}
	
	public String getModel() {
		return model;
	}
	
	public void setModel(String model) {
		this.model = model;
	}
	
	public Engine getEngine() {
		return engine;
	}
	
	@Autowired
	public void setEngine(Engine engine) {
		this.engine = engine;
	}
	
	....
	
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
	xmlns:p="http://www.springframework.org/schema/p"
	xmlns:c="http://www.springframework.org/schema/c"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

  <bean class="org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor" />
  
  //자바코드->  AutowiredAnnotationBeanPostProcessor tempAutowiredPostProcessor=
  				new AutowiredAnnotationBeanPostProcessor();
    
  <bean id="hyundaiEngine" class="exam.test17.Engine">
      <constructor-arg value="Hyundai"/>
  </bean> 
  //자바코드-> Engine hyundaiEngine = new Engine("Hyundai");
  
  <bean id="car1" class="exam.test17.Car">
      <property name="model" value="Sonata"/>
  </bean>
  
  <bean id="car2" class="exam.test17.Car">
      <property name="model" value="Grandeur"/>
  </bean>
  
  //자바코드 ->	Car car1 = new Car();
  //			Car car2 = new Car();
  //			car1.setModel("Sonata");
  //			car2.setModel("Grandeur");
  //			car1.setEngine(hyundaiEngine);
  //			car2.setEngine(hyundaiEngine);
</beans>
```

<br/>
**`@Autowired`의 `required`속성***

프로퍼티(셋터메서드)에 대해 `@Autowired`를 선언하면 해당 프로퍼티는 기본으로 필수 입력 항목이 된다. 만약 프로퍼티에 주입할 의존 객체를 찾을 수 없으면 예외가 발생한다. 이때 `@Autowired`의 `required`속성을 이용하면 필수 입력 여부를 조정할 수 있다. `required`속성을 false로 설정하면 프로퍼티에 주입할 의존객체를 찾지 못하더라도 예외가 발 생하지 않는다.

```java
@Autowired(required=false)
	public void setEngine(Engine engine) {
		this.engine = engine;
	}
```

<br/>

**`@Qualifier`로 주입할 객체를 지정하기**

`@Autowired`는 프로퍼티에 주입할 수 있는 의존객체가 여러개 있을 경우 오류를 발생시키다. 즉 같은 타입의 객체가 여러개 있으면 그 중에서 어떤 객체를 주입해야 할지 알 수 없기 때문이다. 이 경우 애노테이션 `@Qualifier`를 사용하면 손쉽게 해결 가능하다. `@Qualifier`는 빈의 이름(또는 아이디)으로 의존 객체를 지정할 수 있다.

이 애노테이션을 사용하려면 빈의 후 처리기(bean post processor)가 필요하다. 이런식으로 하다보니 애노테이션을 사용할 때마다 그 애노테이션을 처리할 객체를 등록해야 하는 불편함이 발생한다. 이런 불편함을 해소하기 위해 스프링에서는 애노테이션 처리와 관련된 빈(bean post processor)을 한꺼번에 등록할 수 있는 방법을 제공한다. 다음과 같이 간단히 태그를 선언하면 된다.

```xml
<context:annotation-config/>
```
<br/>

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
	xmlns:p="http://www.springframework.org/schema/p"
	xmlns:c="http://www.springframework.org/schema/c"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

  <context:annotation-config />
          
  <bean id="hyundaiEngine" class="exam.test19.Engine">
      <constructor-arg value="Hyundai"/>
  </bean> 

  <bean id="kiaEngine" class="exam.test19.Engine">
      <constructor-arg value="Kia"/>
  </bean> 

  <bean id="car1" class="exam.test19.Car">
      <property name="model" value="Sonata"/>
  </bean>
</beans>
```

```java
public class Car {	
	...
	
	@Autowired(required=false)
	@Qualifier("kiaEngine")
	public void setEngine( Engine engine) {
		this.engine = engine;
	}
	
	...
	
}
```

engine 프로퍼티(setEngine()) 앞에 `@Qualifier` 선언을 했고, 이는 bean.xml에 선언된 엔진 객체 중에서 kiaEngine를 의존 객체로 주입한다.

<br/>

**`@Autowired` + `@Qualifier` = `@Resource`**

JSR-250 명세에 정의된 `@Resource`를 사용하면 더 간단히 특히 의존객체를 지정할 수 있다. 이름으로 의존 객체를 지정할 경우 이 애노테이션을 사용할 것을 권장한다. 단, `@Autowired`는 `required`속성을 사용하여 프로퍼티 값의 필수 여부를 지정할 수 있지만 `@Resource`에는 그런 기능이 없으므로 `@Resource`는 무조건 필수 입력입니다. 해당 프로퍼티에 주입할 의존 객체가 없으면 오류가 발생하므로 의존객체 주입을 선택 사항으로 만들고 싶다면 `@Autowired`를 사용하면된다.

```java
public class Car {
	...
	
	@Resource(name="kiaEngine")
	public void setEngine( Engine engine) {
		this.engine = engine;
	}
	...
}
```

`@Resource`를 선언하고 name 속성에 주입하고자 하는 의존 객체의 이름을 지정한다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
	xmlns:p="http://www.springframework.org/schema/p"
	xmlns:c="http://www.springframework.org/schema/c"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

  <context:annotation-config />
          
  <bean id="hyundaiEngine" class="exam.test20.Engine">
      <constructor-arg value="Hyundai"/>
  </bean> 

  <bean id="kiaEngine" class="exam.test20.Engine">
      <constructor-arg value="Kia"/>
  </bean> 

  <bean id="car1" class="exam.test20.Car">
      <property name="model" value="Sonata"/>
  </bean>
</beans>
```

`@Resource`에 설정한 대로 해당 의존객체(kiaEngine)가 주입된다.

<br/>

### 빈 자동 등록

컴포넌트 스캔과 애노테이션을 이용해 빈을 자동 등록하는 방법은 다음과 같다.

1. 빈생성 대상이 되는 클래스에 `@Component`애노테이션으로 표시한다.
2. 스프링 IoC컨테이너는 `@Component`가 붙은 클래스를 찾아 빈을 생성한다.

스프링에서는 `@Component` 외에 클래스를 관리하고 이해하는데 도움을 주기 위해 클래스의 역할에 따라 붙일 수 있는 애노테이션을 추가로 제공한다. 다음 표는 빈 생성 대상 클래스임을 표시할 때 붙일 수 있는 애노테이션이다.

<br/>

| 애노테이션 | 설명 |
|----------|-------------|
| `@Component` | 빈 생성 대상이 되는 모든 클래스에 대해 붙일 수 있다. |
| `@Repository` | DAO와 같은 퍼시스턴스(persistence) 역할을 수행하는 클래스에 붙인다. |
| `@Service` | 서비스 역할을 수행하는 클래스에 붙인다. |
| `@Controller` | MVC구조에서 Controller 역할을 수행하는 클래스에 붙인다. |

**`@Component`예>**

```java
@Component("car")
public class Car {
	...
	@Autowired
	public void setEngine(Engine engine) {
		this.engine = engine;
	}
	...
}
```

`@Component`에 들어가는 속성값은 빈의 이름이다. getBean()을 호출할 때 사용한다. 


```java
@Component("engine")
public class Engine {
	
	...
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
	xmlns:p="http://www.springframework.org/schema/p"
	xmlns:c="http://www.springframework.org/schema/c"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

  <context:component-scan base-package="exam.test21"/>

</beans>
```

`<context:component-scan/>` 태그를 사용하여 `@Component, @Repository` 등 빈 생성 표시자(애노테이션)가 붙은 클래스를 검색하라고 선언한다.

앞의 코드의 경우 exam.test21 패키지 및 그 하위 패키지를 모두 검색할 것이다.

`<context:component-scan/>` 태그를 선언하면 `<context:annotation-config/>`태그의 기능이 자동으로 활성화된다.

```java
package exam.test21;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {

	public static void main(String[] args) {
		ClassPathXmlApplicationContext ctx = 
				new ClassPathXmlApplicationContext("exam/test21/beans.xml");
		
		Car car = (Car) ctx.getBean("car");
		
		Engine engine = (Engine) ctx.getBean("engine");
		engine.setMaker("Hyundai");
		engine.setCc(1997);
		
		if (car != null) {
			System.out.println("car != null");
		}
		
		if (engine != null) {
			System.out.println("engine != null");
		}
		
		System.out.println(car);
		
		System.out.println(car.toString());
	}
}
/*
car != null
engine != null
[car:null
	[Engine:hyundai,1997]
]
*/
```

이렇게 컴포넌트 자동 탐색(`<context:component-scan/>`)과 빈 자동생성 표시자(`@Component` 등)를 이용하면 빈 설정 파일(bean.xml)에 일일이 빈 정보를 선언할 필요가 없다.

<br/>

>**컴포넌트 탐색의 포함 조건과 제외 조건**
>
>`<context:component-scan/>`을 사용할 때 다음과 같이 탐색 조건을 추가할 수 있다.
>
>```xml
><context:component-scan base-package="exam.test21">
>	<context:include-filter type="" expression=""/>
>	<context:exclude-filter type="" expression=""/>
></context:component-scan>
>```
>
>`context:include-filter`태그는 빈 탐색에 포함할 대상자를 지정할 때사용한다. 
>
>`context:exclude-filter`태그는 빈 탐색에 제외할 대상자를 지정할 때 사용한다.





<br/>

### Reference
* [열혈강의 자바 웹 개발 워크북](http://www.yes24.com/24/Goods/13159413?Acode=101)