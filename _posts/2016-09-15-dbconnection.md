---
layout: post
title: DB커넥션 관련 정리
date: 2016-09-15 20:10
categories: web
---

# 1. DAO(Data Access Object)


DAO는 데이터 처리를 전문으로 하는 객체이다. 데이터베이스나 파일, 메모리 등을 이용하여 애플리케이션 데이터를 생성, 조회, 변경, 삭제하는 역할을 수행한다. DAO는 보통 하나의 DB테이블이나 DB뷰에 대응하고, 여러개의 테이블을 조인(join)한 데이터도 다룬다.


![]({{ site.url }}/images/dao.png)


1. 웹브라우저에서 회원 목록을 요청한다.
2. MemeberListServlet은 회원 목록 데이터를 얻기 위해 MemberDao의 메서드를 호출한다.
3. MemeberDao는 데이터베이스로부터 회원 정보를 가져온다.
4. MemberDao는 겨로가 레코드의 개수만큼 Member객체를 만들어 서블릿에게 반환한다.
5. MemberListServlet은 MemberDao로 부터 받은 회원 목록을 MemberList.jsp에 전달한다.
6. MemberList.jsp는 회원 목록 데이터를 가지고 화면을 생성한다. 그리고 제어권을 MemberListServlet에게 넘긴다.
7. MemberListServlet에게 넘긴다.


**예>**

```java
public class MemberDao {
  Connection connection;

  public void setConnection(Connection connection) {
    this.connection = connection;
  }

  public List<Member> selectList() throws Exception {
    Statement stmt = null;
    ResultSet rs = null;

    try {
      stmt = connection.createStatement();
      rs = stmt.executeQuery(
          "SELECT MNO,MNAME,EMAIL,CRE_DATE" + 
              " FROM MEMBERS" +
          " ORDER BY MNO ASC");

      ArrayList<Member> members = new ArrayList<Member>();

      while(rs.next()) {
        members.add(new Member()
        .setNo(rs.getInt("MNO"))
        .setName(rs.getString("MNAME"))
        .setEmail(rs.getString("EMAIL"))
        .setCreatedDate(rs.getDate("CRE_DATE"))	);
      }

      return members;

    } catch (Exception e) {
      throw e;

    } finally {
      try {if (rs != null) rs.close();} catch(Exception e) {}
      try {if (stmt != null) stmt.close();} catch(Exception e) {}
    }
  }
} 
```

위의 예에서 두가지를 배울 수 있다. 


* 첫번째는 `프로그래밍의 유연성`이다. `selectList()`에서 리턴 타입이 List인터페이스인 것이다. 하지만 실제 리턴값은 ArrayList객체이다. 이런것을 프로그래밍의 유연성이라고 한다. 

* 두번째는 `의존성주입(DI, Dependency Injection)` 다른말로 `역제어(IoC, Inversion of Control)`이다. connection 인스턴스 변수와 셋터 메서드는 MemberDao에서 ServletContext에 접근할 수 없기 때문에, ServletContext에 보관된 DB Connectoin 객체를 꺼낼 수 없어 필요하다. 외부로 부터 Connection객체를 connection 인스턴스변수와 셋터 메서드에 주입한다. 이런 것을 의존성 주입 또는 역제어라고 한다.<br />


# 2. Servlet - Listener

서블릿 컨테이너는 웹 어플리케이션의 상태를 모니터링 할 수 있도록 웹어플리케이션의 시작에서 종료까지 주요한 사건에 대한 알람기능을 제공한다. 이를 사용하기 위해 규칙에 따라 객체를 만들어 DD파일에 등록하면 된다.



| 사건 | 인터페이스 |
|------------|------------------|
| 웹어플 : 시작하거나 종료할때 | javax.servlet.ServletContextListener |
| 웹어플 : ServletContext에 값을 추가하고, 제거하고, 대체할 때 | javax.servlet.ServletContextAttributeListener |
| 세션 : 생성,소멸할때 | javax.servlet.http.HttpSessionListener |
| 세션 : 활성, 비활성 할때 | java.servlet.http.HttpSessionActivationListener |
| 세션 : HttpSession에 값을 추가하고, 제거하고, 대체할 때 | javax.servlet.http.HttpSessionAttributeListener |
| 요청을 받고 응답할때 | javax.servlet.ServletRequestListener |
| ServletRequest에 값을 추가하고, 제거하고, 대체할 때 | javax.servlet.ServletRequestAttributeListener |


위의 표는 다양한 이벤트 리스너를 정리를 하였다.

DAO의 경우처럼 여러 서블릿이 사용하는 객체는 서로 공유하는 것이 메모리 관리나 실행 속도 측면에서 좋다고 한다. DAO를 공유하려면 ServletContext에 저장하는 것이 좋다. ServletContext는 웹 어플리케이션이 종료될 때까지 유지되는 보관소이기 때문이다.


### ServletContextListener의 활용


![]({{ site.url }}/images/servletcontextlinstener.png)


1. 웹어플리케이션이 시작되면, 서블릿 컨테이너는 ServletContextListener의 구현체에 대해 contextInitialized() 메서드를 호출한다.
2. 리스너는 DB커넥션 객체를 생성한다.
3. 리스너는 DAO 객체를 생성한다.
4. DB커넥션 객체를 DAO에 주입한다.
5. 서블릿들이 DAO 객체를 사용할 수 있도록 ServletContext보관소에 저장한다.
6. 만약 웹어플리케이션이 종료되면, 서블릿 컨테이너는 리스너의 contextDestroyed()메서드를 호출한다.


**ServletContextListener 인터페이스 계층도**

* javax.servlet.**ServeltContextListener**
	* contextInitialized()
	* contextDestroyed()


`contextInitialized()`는 웹어플리케이션이 시작될 때 호출된다. 공용객체를 준비해야한다면, 여기에 작성하면된다.

`contextDestroyed()`는 웹어플리케이션이 종료되기 전에 호출된다. 자원 해제를 해야 한다면, 여기게 작성하면된다.



#### ContextLoaderListener의 배치

역시나 두가지 방법이 있다. 첫번째는 애노테이션을 사용하는 방법이다. 클래스 선언 위에 `@WebListener`라고 애노테이션을 붙이면 된다.


**예>**

```java
@WebListener
public class ContextLoaderListener ----
```

두번때는 DD파일에 XML태그를 선언하는 것이다. `<web-app>`태그 안에 작성 하면되고 Servlet 2.4버전까지는 `<listener>` 태그를 작성할 때 순서를 지켜야한다. 즉 `<filter-mapping>`태그 다음에, `<servlet>`태그 이전에 와야한다. Servlet 2.5 버전 부터는 순서에 상관없다.

**예>**

```java
<listener>
	<listener-class>spms.listeners.ContextLoaderListener</listener-class>
</listener>
```

이렇게 ServletContext에 들어 있는 MemberDao객체를 재사용하게 되면, 객체를 생성할 필요가 없기 때문에 실행 속도가 빨라지고, 가비지(garvage)가 발생하지 않는다.

<br />


# 3. DB 커넥션풀

자주 쓰는 객체를 미리 만들어 두고, 필요할 때마다 빌리고, 사용한 다음 반납하는 방식을 `풀링(pooling)`이라고 한다. 이렇게 여러 개의 객체를 모아 둔것을 `객체 풀(object pool)`이라고 하고, 여러 개의 DB커넥션을 관리하는 객체를 `DB 커넥션풀`이라고 한다.

커넥션 풀에 대한 필요성은 이전 처럼 하나의 커넥션에서 공유하여 사용할 경우 롤백(rollback)이 발생할 경우 다른 요청에 대한 작업 내용까지 모두 롤백을 해야하는 치명적인 문제가 있기 때문에 필요하다. SQL작업을 할 때마다 DB 커넥션 객체를 생성한다면, 실행속도가 느려지고 많은 가비지가 생선된다. 

커넥션 객체를 생성하는데 속도가 느려지는 이유는, 커넥션을 맺을 때마다 데이터베이스 서버는 사용자 인증과 권한 검사를 수행하고 요청 처리를 위한 준비 작업을 해야하기 때문이다.


![]({{ site.url }}/images/connectionpool.png)


#### DataSourc 와 JNDI(Java Naming and Directory Interface)


DataSource는 JDBC확장 API를 정으한 javax.sql패키지에 들어 있다. DriverManager를 통해 DB 커넥션을 얻는 것보다 더 좋은 기법을 제공한다. 특히 다음 두가지 점에서 좋다.

첫째, DataSource는 서버에서 관리하기 때문에 데이터베이스나 JDBC 드라이버가 변경되더라도 어플리케이션을 바꿀 필요가 없다.

둘째, DataSource를 사용하면 Connection과 Statement객체를 풀링할 수 있으며, 분산 트랜잭션을 다룰 수 있다.

Datasource를 사용하려면 javax.sql패키지의 구현체가 필요하다. 이전버전 commons-dbcp-1.4jar, commons-pool-1.6.jar, commons-collections-3.2.1-bin.zip 3개의 라이브러리는 톰캣 6.0 부터 tomcat-dbcp.jar 파일로 하나로 통합되었다.

아파치 DBCP 라이브러리에서 DataSource인터페이스를 구현한 클래스는 BasicDataSource이다. 아래 소스는 구현 예시이다.
<br />



```java
//ServletContextListener 클래스

@WebListener
public class ContextLoaderListener implements ServletContextListener {
  BasicDataSource ds;
  
  @Override
  public void contextInitialized(ServletContextEvent event) {
    try {
      ServletContext sc = event.getServletContext();
      
      ds = new BasicDataSource();
      ds.setDriverClassName(sc.getInitParameter("driver"));
      ds.setUrl(sc.getInitParameter("url"));
      ds.setUsername(sc.getInitParameter("username"));
      ds.setPassword(sc.getInitParameter("password"));
      
      MemberDao memberDao = new MemberDao();
      memberDao.setDataSource(ds);
      
      sc.setAttribute("memberDao", memberDao);

    } catch(Throwable e) {
      e.printStackTrace();
    }
  }

  @Override
  public void contextDestroyed(ServletContextEvent event) {
    try { if (ds != null) ds.close(); } catch (SQLException e) {}
  }
}
```

```java
//DAO 클래스

public class MemberDao {
  DataSource ds;

  public void setDataSource(DataSource ds) {
    this.ds = ds;
  }
   public List<Member> selectList() throws Exception {
    Connection connection = null;
    Statement stmt = null;
    ResultSet rs = null;
    
    try {
      connection = ds.getConnection();
      stmt = connection.createStatement();
      rs = stmt.executeQuery(
          "SELECT MNO,MNAME,EMAIL,CRE_DATE" + 
              " FROM MEMBERS" +
          " ORDER BY MNO ASC");

      ArrayList<Member> members = new ArrayList<Member>();

      while(rs.next()) {
        members.add(new Member()
        .setNo(rs.getInt("MNO"))
        .setName(rs.getString("MNAME"))
        .setEmail(rs.getString("EMAIL"))
        .setCreatedDate(rs.getDate("CRE_DATE"))	);
      }

      return members;

    } catch (Exception e) {
      throw e;

    } finally {
      try {if (rs != null) rs.close();} catch(Exception e) {}
      try {if (stmt != null) stmt.close();} catch(Exception e) {}
      try {if (connection != null) connection.close();} catch(Exception e) {}
    }
  }
}
```

<br />


#### DataSource의 Connection


DataSource가 만들어 주는 Connection 객체는 DriverManager가 만들어 주는 커넥션 객체를 한번 더 포장한 것이다.

1. MemberDao가 DataSource에게 커넥션을 달라고 요청을한다.
2. DataSource sms DriverManager가 생성한 커넥션을 리턴하는 것이 아니라, 커넥션 대행 객체(Proxy Object)를 리턴한다. 아파치 DBCP 컴포넌트의 BasicDataSource를 사용할 경우, `PoolableConnection`객체를 반환한다. 바로 이객체가 커넥션의 대행 객체이다. 이 대행 객체는 진짜 커넥션을 가리키는 참조 변수(\_conn)와 커넥션풀을 가리키는 참조변수(\_pool)가 들어 있다. 물론 대행 객체도 Connection처럼 보이기 위해, java.sql.Connection 인터페이스를 구현했다. 구현하엿다는 의미는 외부에서 요청이 들어왔을 때, 자신이 직접 실행한다는 것이 아니라 진짜 커넥션에게 위임한다는 의미이다.


#### 톰캣 서버에 DataSource 설정하기

DataSource를 사용하는 이유는 서버에서 관리하기 때문에 데이터베이스에 대한 정보가 바뀌거나 JDBC드라이버가 교체되더라도 어플리케이션에 영향을 주지 않기 때문이라고 했다. 그러나 앞의 예제에서는 `BasicDataSource`를 사용하기 때문에 진정으로 DataSource의 이점을 누릴 수 없다. 그래서 DataSource를 서버에 적용한다. 


>서버 제품에 따라 DataSource를 설정하는 방법이 다름


1. 톰켓 실행 환경 폴더에서 context.xml 파일을 찾는다.

	![]({{ site.url }}/images/contextxml.png)

2. context.xml 파일을 열고 다음과 같이 편집을 한다.

	```java
	<?xml version="1.0" encoding="UTF-8"?>
	<Context>
    <WatchedResource>WEB-INF/web.xml</WatchedResource>
	<Resource name="jdbc/studydb" auth="Container" type="javax.sql.DataSource"
	    maxActive="10" maxIdle="3" maxWait="10000" 
	    username="study"
	    password="study" 
	    driverClassName="com.mysql.jdbc.Driver"
	    url="jdbc:mysql://localhost/studydb" 
	    closeMethod="close"/>
	
	</Context>
	```
	
	`<Context>`태그 안에 `<Resource>` 태그를 추가한다.
	
	**\<Resource\>태그의 속성들**


	| 속성명 | 설명 |
	|:---:|----|
	| name | JNDI이름. Context의 lookup()를 사용하여 자원을 찾을 때 사용한다. `java:comp/env`디렉토리에서 찾을 수 있다. |
	| auth | 자원 관리의 주체를 지정한다. 설정 가능한 값으로 Application 또는 Container가 가능하다. |
	| type | 자원의 타입을 지정한다. 패키지 이름을 포함한 클래스 이름(fully-qualified Java class name)이어야 한다. |
	| driverClassName | JDBC 드라이버 클래스의 이름. 패키지 이름을 포함해야한다. |
	| url | 데이터베이스 커넥션 URL |
	| username | 데이터베이스 사용자 이름 |
	| password | 데이터베이스 사용자의 암호 |
	| maxActive | DataSource로부터 꺼낼 수 있는 커넥션의 최대 개수. 기본값 8개 |
	| maxIdle | DataSource에서 유지할 수 있는 사용되지 않는 커넥션의 최대 개수. 최대 유지개수를 넘어서 반납되는 커넥션은 닫아 버린다. 기본값 8개 |
	| maxWait | 발급한 커넥션의 수가 최댓값에 도달한 상태에서 또다시 커넥션을 달라는 요청이 들어 왔을 때, 커넥션을 준비하기 위해 기다리는 최대 밀리초. 최대 밀리초가 지날 때까지 반납되는 커넥션이 없으면 예외를 던진다. 기본값은 -1. 즉 커넥션을 반납할 때까지 기다린다. |
	| closeMethod | 톰켓 서버가 종료될 때, 자원을 해제하기 위해 호출되는 메서드의 이름이다.단 매개변수가 없는 메서드여야 한다. 톰캣 서버는 내부적으로 DataSource를 생성할 때 아파치 DBCP의 BasicDataSource 구현체를 사용한다. BasicDataSource의 자원 해제 메서드는 close()이기 때문에, 이 속성의 값으로 close를 지정하면 된다. |


3. 웹어플리케이션의 DD파일을 열고 다음과 같이 편집한다.
	
	```java
	<?xml version="1.0" encoding="UTF-8"?>
	<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		xmlns="http://java.sun.com/xml/ns/javaee"
		xsi:schemaLocation="http://java.sun.com/xml/ns/javaee 
		http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd" 
		id="WebApp_ID" version="3.0">
		<display-name>web05</display-name>
		
  		<resource-ref>
      		<res-ref-name>jdbc/studydb</res-ref-name>
      		<res-type>javax.sql.DataSource</res-type>
      		<res-auth>Container</res-auth>
  		</resource-ref>
		
		<welcome-file-list>
			<welcome-file>index.html</welcome-file>
			<welcome-file>index.htm</welcome-file>
			<welcome-file>index.jsp</welcome-file>
		</welcome-file-list>
	</web-app>	
	```
	
	
	`<web-app>`태그 안에 `<resource-ref>`태그 문법은 다음과 같다.
	
	
	
	```
	<resource-ref>
		<res-ref-name>JNDI 이름</res-ref-naem>
		<res-type>리턴될 자원의 클래스 이름(패키지명포함)</res-type>
		<res-auth>자원 관리의 주체</res-auth>
	</resource-ref>
	```

4. 클래스에 적용하기
	
	
	```java
	import javax.naming.InitialContext;
	import javax.servlet.ServletContext;
	import javax.servlet.ServletContextEvent;
	import javax.servlet.ServletContextListener;
	import javax.servlet.annotation.WebListener;
	import javax.sql.DataSource;
	
	import spms.dao.MemberDao;
	
	@WebListener
	public class ContextLoaderListener implements ServletContextListener {
  	@Override
  	public void contextInitialized(ServletContextEvent event) {
    	try {
      	ServletContext sc = event.getServletContext();
      	
      	InitialContext initialContext = new InitialContext();
      	DataSource ds = (DataSource)initialContext.lookup(
        	  "java:comp/env/jdbc/studydb");
      		
      	MemberDao memberDao = new MemberDao();
      	memberDao.setDataSource(ds);
      	
      	sc.setAttribute("memberDao", memberDao);
	
    	} catch(Throwable e) {
    	  e.printStackTrace();
  	  	}
  	}
	
  	@Override
 	public void contextDestroyed(ServletContextEvent event) {}
	}
	```
	
	
	톰캣서버에서 자원을 찾기 위해 `InitialContext`객체를 생성햇다. `InitialContext`의 	`lookup()`메서드를 이용하면, JNDI 이름으로 등록되어 있는 서버 자원을 찾을 수 있다. 	`contextDestroyed()`메서드를 보면 자원을 해제하는 어떤 코드도 작성되어 있지 않다. 이는 톰	캣서버에 자동으로 해제하라고 설정되어 있기 때문dlek.
	
<br />


>**JNDI란**
>
>JNDI는 Java Naming and Directory Interface API의 머리 글자이다. 디렉터리 서비스에 접근하는데 필요한 API이며 이 API를 사용하여 서버의 자원을 찾을 수 잇다. 여기서 자원은 데이터베이스 서버나 메시징 시스템과 같은 다른 시스템과의 연결을 제공하는 객체이다. 특히 JDBC 자원을 데이터 소스라고 한다.
>
>자원을 서버에 등록할 때는 고유한 JNDI이름을 붙인다. JNDI 이름은 사용자에게 친숙한 디렉터리경로 형태를 가진다. 
>
>따라서 'jdbc/mydb'라는 데이터 소스가 있어서 서버에서 이 자원을 찾으려면 'java:comp/env/jdbc/mydb' JNDI이름으로 찾아야한다.
>
>>
>>| java:comp/env | 응용프로그램 환경 항목 |
|---------------------------|------------------------|
| java:comp/env/jdbc | JDBC데이터소스 |
| java:comp/ejd | EJB컴포넌트 |
| java:comp/UserTransaction | UserTransaction 객체 |
| java:comp/env/mail | JavaMail연결객체 |
| java:comp/env/url | URL정보 |
| java:comp/env/jms | JMS연결 객체 |

<br />






### Reference
* [열혈강의 자바 웹 개발 워크북](http://www.yes24.com/24/Goods/13159413?Acode=101)
* [장인개발자를 꿈꾸는 :: 기록하는 공간](http://devbox.tistory.com/entry/JSP-커넥션-풀-1)