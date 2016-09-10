---
layout: post
title: Servlet와 JDBC
date: 2016-09-10 21:00:18
categories: web
---

# 1. JDBC종류
DBMS 밴더는 자신들의 DBMS에 접속할 수 있는 API를 만들어 배포한다. 즉 DMBS를 사용하기 위해서는 밴더가 제공하는 API에 맞게 개발해야한다. 문제는 밴더들 별로, API가 달라 개발비용 및 유지보수에 많은 비용이 들어간다. 

그래서 마이크로소프트에서 밴더에 종속 받지 않고 DBMS에 접근 가능한 ODBC를 만들어 배포했다. 대부분의 DBMS API는 ODBC에 포함되어 있다. 이 라이브어리를 `ODBC 드라이버`라고 명칭한다. 

## 1) JDBC Type 1 

![]({{ site.url }}/images/jdbctype1.png)

주요특징은 다음과 같다.

- ODBC 드라이버를 사용하여 DBMS에 접속
- JRE에 포함되어 있다
- 속도가 유형중에서 가장 느리다.
- Excel이나 Access파일에 접속할 때 유용하다.

## 2) JDBC Type 2

![]({{ site.url }}/images/jdbctype2.png)

주요특징은 다음과 같다.

- DBMS밴더 API 사용
- 별도 JDBC 드라이버를 내려받아야함
- DBMS 클라이언트의 설치 필요

## 3) JDBC Type 3

![]({{ site.url }}/images/jdbctype3.png)

주요특징은 다음과 같다.

- 미들웨어 서버를 경유하여 DBMS에 접속
- 미들웨어에서 제공하는 JDBC 드라이버 사용
- ODBC나 밴더 API처럼 C나 C++로 만든 API를 호출하지 않기 때문에 Pure Java라 한다.

## 4) JDBC Type 4

![]({{ site.url }}/images/jdbctype4.png)

주요특징은 다음과 같다.

- DBMS 프로토콜을 사용하여 직접 통신(ODBC드라이버가 필요없다.)
- 별도로 JDBC 드라이버 내려받아야한다.
- Pure Java
- 주로 사용되는 드라이버이다.

# 2. JDBC와 서블릿

## 1) java.sql

### 1-1) 드라이버등록
JDBC프로그래밍의 첫번째 작업은 `DriverManager`를 이용하여 java.sql.Driver 인터페이스 구현체를 등록하는 것이다. 방법은 두가지가 있다.

```java
DriverManager.registerDriver(new com.mysql.jdbc.Driver()); //DriverManager를 직접이용

Class.forName(this.getInitParameter("QName")); //클래스로딩 

//QName을 서블릿 초기화 매개변수의 값으 가져와서 작성하는 방법도 있다.

this.getInitParameter(/*매개변수 이름 */);
```

### 1-2) 데이터베이스 연결

다음과 같이 `DriverManager`의 `getConnection()`을 호출하여 MySQL 서버에 연결할 수 있다.

```java
conn = DriverManager.getConnection(
					"jdbc:mysql://localhost/studydb", //JDBC URL
					"study",	// DBMS 사용자 아이디
					"study");	// DBMS 사용자 암호
```

### 1-3) SQL 실행 및 준비

```java
stmt = conn.createStatement(); //java.sql.Statement 인터페이스

stmt = conn.prepareStatement(       //java.sql.PrepareStatement 인터페이스
					"INSERT INTO MEMBERS(EMAIL,PWD,MNAME,CRE_DATE,MOD_DATE)"
					+ " VALUES (?,?,?,NOW(),NOW())");
			stmt.setString(1, request.getParameter("email"));
			stmt.setString(2, request.getParameter("password"));
			stmt.setString(3, request.getParameter("name"));

```

#### java.sql.Statement 인터페이스의 구현체

메서드|설명|
------|----|
executeQuery()|결과가 만들어지는 SQL문을 실행할때 사용, 보통 SELECT문을 실행|
executeUpdate()|DML과 DDL관련 SQL문을 실행할때 사용|
execute()|SELECT, DML, DDL 명령문 모두에 사용가능|
executeBatch()|addBatch()로 등록한 여러개의 SQL문을 한꺼번에 실행할때 사용|


`PreparedStatement`는 반복적인 질의를 하거나, 입력 매개변수가 많은 경우에 유용하다. 특히 이미지와 같은 바이너리 데이터를 저장하거나 변경할 경우는 `PreparedStatement`만 가능하다.

위의 예제에서 SQL문에서 `?`문자로 표시된 것을 입력 매개변수라고 한다. 입력 항목의 값은 SQL문을 실행하기 전에 `setXXX()`메서드를 호출하여 설정해야한다.

|비교항목|Statement|PreparedStatement|
|---------|---------|-----------------|
|실행속도|질의 할때마다 SQL문을 컴파일한다.|SQL문을 미리 준비하여 컴파일 해둔다. 입력 매개변수 값만 추가하여 서버에 전송한다. 특히 여러번 반복하여 질의하는 경우, 실행속도가 바름|
|바이너리데이터전송|불가능|가능|
|프로그래밍 편의성|SQL문 안에 입력 매개변수 값이 포함되어 있어서 SQL문이 복잡하고 매개변수가 여러개인 경우 코드 관리가 힘들다.|SQL문과 입력 매개변수가 분리되어 있어 코드작성이 편리하다.|

#### java.sql.ResultSet 인터페이스의 구현체
|메서드     |설명                                                      |
|-----------|----------------------------------------------------------|
|first()    |서버에서 첫번째 레코드를 가져온다.                        |
|last()     |서버에서 마지막번째 레코드를 가져온다.                    |
|previous() |서버에서 이전 레코드를 가져온다.                          |
|next()     |서버에서 다음 레코드를 가져온다.                          |
|getXXX()   |레코드에서 특정 컬럼의 값을 꺼낸다. XXX는 칼럼의 타입이다.|
|updateXXX()|레코드에서 특정 칼럼의 값을 변경한다.                     |
|deleteRow()|현재 레코드를 지운다.                                     |



### 1-3) JDBC 프로그래밍의 마무리

JDBC 프로그래밍을 할 때 주의할 점은 정상적으로 수행되는 오류가 발생하든 간에 반드시 자원해제를 수행하는 것이다. 자원을 해제하기 가장 좋은 위치는 `finally` 블록이다.

```java
finally {
			try {if (rs != null) rs.close();} catch(Exception e) {}
			try {if (stmt != null) stmt.close();} catch(Exception e) {}
			try {if (conn != null) conn.close();} catch(Exception e) {}
		}
```

## 2) HttpServlet

### 2-1) 절대경로, 상대경로

`<a>`태그의 링크 URL이 `/`로 시작하면 절대 경로이고, `/` 시작하지 않으면 상대 경로이다.

`절대경로`는 웹 서버 루트를 기준으로 계산한다.

예) 
>서버루트 --> http://localhost:8080
>
>최종경로 --> http://localhost:8080/web22/member/add
>
>절대경로 --> /web22/member/add


`상대경로`는 현재 경로를 기준으로 계산한다. 

예)
>현재경로 --> http://localhost:8080/web/member/list
>
>최종경로 --> http://localhost:8080/web/member/add
>
>상대경로 --> add

### 2-2) HttpServelt

![]({{ site.url }}/images/httpservlet.png)

`HttpServlet`클래스는 `GenericServlet`클래스의 하위 클래스이다. `GenericServlet`클래스를 상속 받는 것과 마찬가지로 `javax.servlet.Servlet`인터페이스를 구현한 것이된다.

서블릿 컨테이너는 Servlet규칙에 정의된 메서드를 호출하기 때문에, 서블릿 객체는 반드시 Servelt인터페이스를 구현 해야한다.

![]({{ site.url }}/images/httpservletservice.png)

`HttpServlet`클래스는 클라이언트 요청이 들어오면, 첫째로 상속받은 `service()`메서드가 호출되고, 둘째로 `service()`는 클라이언트 요청 방식에 따라 `doGet()`이나 `doPost()` 등의 메서드를 호출한다. 따라서 `HttpServlet`을 상속받을 때 `service()`메서드를 직접 구현하기보다는 클라이언트의 요청방식에 따라 `doXXX()`메서드를 오버라이딩 한다.


## 3) Refresh - Redirect

Refresh는 잠간이나마 작업 결과를 출력하고 다른 페이지로 이동하기를 윈할때 사용하고, Redirect는 작업결과를 출력하지 않고 즉시 다른페이지로 이동하기를 원할때 상용한다.
### 3-1) 응답 헤더를 이용한 Refresh

```java
response.addHeader("Refresh","1;url=list");
```
1초후에 list페이지로 이동한다.

### 3-2) Html의 meta태그를 이용한 Refresh

```java
out.println("<meta http-equiv='refresh' content='1; url=list'>");
```
1초후에 list페이지로 이동한다.


### 3-3) redirect메서드 sendRedirect()

```java
response.sendRedirect("list");
```
list페이지로 redirection

## 4) Context 매개변수

```xml
<!-- 컨텍스트 초기화 파라미터 -->
	<context-param>
		<param-name>driver</param-name>
		<param-value>com.mysql.jdbc.Driver</param-value>
	</context-param>
	<context-param>
		<param-name>url</param-name>
		<param-value>jdbc:mysql://localhost/studydb</param-value>
	</context-param>
	<context-param>
		<param-name>username</param-name>
		<param-value>study</param-value>
	</context-param>
	<context-param>
		<param-name>password</param-name>
		<param-value>study</param-value>
	</context-param>
```
DD파일에 컨텍스트 초기화 파리마터를 정의하고 사용은 다음과 같이한다.

```java
ServletContext sc = this.getServletContext();	//HttpServlet으로 부터 상속받음
			Class.forName(sc.getInitParameter("driver"));  
			conn = DriverManager.getConnection(
						sc.getInitParameter("url"),
						sc.getInitParameter("username"),
						sc.getInitParameter("password"));
						
			//getInitParameter()를 호출하여 사용 
```

## 5) Filter

필터는 서블릿 실행 전후에 어떤 작업을 하고자 할때 사용하는 기술이다. 예를 들면 클라이언트가 보낸 데이터의 암호를 해제한다거나, 서블릿이 실행되기 전에 필요한 자원을 미리 준비한다거나, 서블릿이 실행될 때마다 로그를 남긴다거나 하는 작업을 필터를 통해 처리할 수 있다.

>java.servlet --> Filter 
>>`init()` 
>>`doFilter()`
>>`destory()`

- `init()` 메서드는 필터 객체가 생성되고 나서 준비 작업을 위해 딱한번 호출된다. 이 메서드의 매개변수는 `FilterConfig`객체이다.
- `doFilter()` 필터와 연결된 URL에 대해 요청이 들어오면 `doFilter()`가 항상 호출된다. 이메서드에 필터가 할 일을 작성하면된다.

```java
	@Override
	public void doFilter(
			ServletRequest request, ServletResponse response,
			FilterChain nextFilter) throws IOException, ServletException {
			
		request.setCharacterEncoding(config.getInitParameter("encoding"));
		
		// 다음 필터를 호출, 더이상 필터가 없다면 서블릿의 service()가 호출됨.
		nextFilter.doFilter(request, response);
		
	}
```
- `destroy()`서블릿 컨테이너는 웹 어플리케이션을 종료하기 전에 필터들에 대해 `destroy()`를 호출하여 마무리 작업을 할 수 있는 기회를 준다.

#### 필터 배치

필터의 배치 방법도 서블릿과 마친가지로 두가지 가 있다. DD파일에 기술하는 방법과 애노테이션을 기술하는 방법이 있다.

DD파일에 작서한 예)

```xml
<!-- 필터 선언 --> 
	<filter>
		<filter-name>CharacterEncodingFilter</filter-name>
		<filter-class>spms.filters.CharacterEncodingFilter</filter-class>
		<init-param>
			<param-name>encoding</param-name>
			<param-value>UTF-8</param-value>
		</init-param>
	</filter>
	
	<!-- 필터 URL 매핑 -->
	<filter-mapping>
		<filter-name>CharacterEncodingFilter</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>
```

애노테이션을 이용한 필터 배치 예)

```java
@WebFilter(
	urlPatterns="/*",
	initParams={
		@WebInitParam(name="encoding",value="UTF-8")
	})
public class CharacterEncodingFilter implements Filter{
	FilterConfig config;
	
	@Override
	public void init(FilterConfig config) throws ServletException {
		this.config = config;
	}
	@Override
	public void doFilter(
			ServletRequest request, ServletResponse response,
			FilterChain nextFilter) throws IOException, ServletException {
		request.setCharacterEncoding(config.getInitParameter("encoding"));
		nextFilter.doFilter(request, response);
	}
	@Override
	public void destroy() {}
}
```

### Reference
- [열혈강의 자바 웹 개발 워크북](http://www.yes24.com/24/Goods/13159413?Acode=101)
