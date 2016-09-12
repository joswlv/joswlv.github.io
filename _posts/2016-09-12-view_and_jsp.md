---
layout: post
title: 뷰 컴포넌트와 JSP
date: 2016-09-12 19:50
categories: web
---

# 1. JSP(Java Server Pages)

JSP 기술의 가장 중요한 목적은 콘텐츠를 출력하는 코딩을 단순화 하는 것이다.

### JSP 구동원리

1. 개발자는 서버에 JSP 파일은 작성한다. 클라이언트가 JSP를 실행해 달라고 요청을 하면, 서블릿 컨테이너는 JSP파일에 대응하는 자바 서블릿을 찾아서 실행한다.
2. 만약 JSP에 대응하는 서블릿이 없거나 JSP파일이 변경되었다면, `JSP엔진`을 통해서 JSP파일을 해석하여 서블릿 자바소스를 생성한다.
3. 서블릿 자바 소스는 자바 컴파일러를 통해 서블릿 클래스 파일로 컴파일된다. JSP파일을 바꿀 때마다 이 과정을 반복한다.
4. JSP로부터 생성된 서블릿은 서블릿 구동 방식에 다라 실행된다. 즉 서블릿의 `service()`메서드가 호출되고, 출력 메서드를 통해 서블릿이 생성한 HTML 화면을 웹브라우저로 보낸다.


> 결론은 JSP가 직접 실행되는 것이 아니라 JSP로부터 만들어진 서블릿이 실행된다.

### HttpJspPage 인터페이스

JSP엔진은 JSP 파일로 부터 서블릿 클래스를 생성할 때 `HttpJspPage` 인터페이스를 구현한 클래스를 만든다. 

상속관계는 다음과 같다.


> * Servlet(Interface)
> 	* init()
> 	* service()
> 	* destory()
> 	* getServletConfig()
> 	* getServletInfo()
> 
> * JspPage(Interface)
> 	* jspInit()
> 	* jspDestory()
> 
> * HttpJspPage(Interface)
> 	* _jspSerivce()
> 


**결론적으로 HttpJspPage는 Servlet 인터페이스도 구현한다는 것이기 때문에 해당 클래스는 서블릿이 될 수 있다.**


* jspInit()
	 
	 * JspPage에 선언된 jspInit()는 JSP객체가 생성될 때 호출된다. 자동 생성된 서블릿 소스 코드에서 init()가 호출될 때 jspInit()를 호출하도록 코딩되어 있다. 그래서 JSP 페이지에서 init()를 오버라이딩 할 일이 있다면 init() 대신 jspInit()를 오버라이딩하면된다.

* jspDestory()

	* jspInit()과 마찬가지로 JSP페이지에서 destory()를 오버라이딩 할 일이 있으면 jspDestory()를 오버라이딩 해서 사용하면된다.
	
*  _jspService()

	* HttpJspPage에 선언된 _jspService()는 JSP 페이지가 해야 할 작업이 들어 있는 메서드이다. 서블릿 컨테이너가 service()를 호출하면 service() 메서드 내부에서는 바로 이 메서드를 호출함으로써 JSP 페이지에 작성했던 코드가 실행된다.

	
### JSP의 주요 구성 요소

#### 지시자 태그

`page`지시자는 JSP페이지와 관련된 속성을 정의할 때 사용하는 태그이다. JSP 지시자에는 `page`, `taglib`, `include`가 있다.

```java
<%@page import="spms.vo.Member"%>
<%@page import="java.util.ArrayList"%>
<%@ page 
	language="java" 
	contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
```

* `language속성`은 스크립트릿(Scriptlet)이나, 표현식(Expression element), 선언부(Declaration element)를 작성할때 사용한다.
* `contenttype속성`은 출력할 데이터의 MIME타입과 문자 집합을 지정한다. 
* `pageEncoding속성`은 출력할 데이터의 문자 집합을 지정한다. 기본값은 `ISO-8859-1`이다. 만약 이속성을 생략하면 `contentType`에 설정된 값을 사용한다. `contentType속성`에도 없다면 기본값 `ISO-8859-1`을 사용한다.


#### 스크립트릿 태그

JSP 페이지 안에서 자바코드를 넣을 때는 스크립트릿(Scriptlet Elements)태그 `<% %>`안에 작성한다. `<% 자바 코드 %>`

```java
<%
String v1 = "";
String v2 = "";
String result = "";
String[] selected = {"", "", "", ""};

//값이 있을 때만 꺼낸다.
if (request.getParameter("v1") != null) {
	v1 = request.getParameter("v1");
	v2 = request.getParameter("v2");
	String op = request.getParameter("op");
	
	result = calculate(
				Integer.parseInt(v1), 
				Integer.parseInt(v2), 
				op);
	
	if ("+".equals(op)) {
		selected[0] = "selected";
	} else if ("-".equals(op)) {
		selected[1] = "selected";
	} else if ("*".equals(op)) {
		selected[2] = "selected";
	} else if ("/".equals(op)) {
		selected[3] = "selected";
	}
}
%> 
```

위 코드의 자바 서블릿 소스에서 _jspService() 메서드를 보면 스크립트릿 태그안의 자바소스가 그대로 입력된 것을 알 수 있다.

#### JSP 내장 객체

JSP페이지에서 스크립트릿 `<% %>`이나 표현식 `<%= %>`을 작성할 때 별도의 선언 없이 사용하는 자바 객체를 JSP 내장객체(Implicit Objects)라 한다. `_jspService()`는 `javax.servlet.jsp.HttpJspPage` 인터페이스에 선언된 메서드이다. JSP페이지로 서블릿을 만들때 반드시 이 인터페이스를 구현하도록 정의 되어 있다. 

JSP 기술 사양서는 JSP페이지 작성자가 별도 선언 없이 즉시 이용할 수 있는 9개 객체를 정의 한다. 

`request`, `response`, `pageContext`, `session`, `application`, `config`, `out`, `page`, `exception` 객체가 `_jspservice()`메서드에 선언되어 있어 별도의 선언 없이 스크립트릿과 표현식에 JSP내장객체를 사용할 수 있다.

#### 선언문 태그

JSP 선언문(Declarations) `<%! %>`은 서블릿 클래스의 맴버(변수나 메서드)를 선언 할 때 사용하는 태그이다. JSP페이지에서 선언문 `<%! %>`을 작성하는 위치는 위, 아래, 가운데 어디든 상관 없다. 왜나햐면 선언문은 `_jspService()`메서드 안에 복사되는 것이 아니라, `_jspService()`밖의 클래스 블록 안에 복사 되기 때문이다. 

`<%! 맴버 변수 및 메서드 선언 %>`


```java
<%! 
private String calculate(int a, int b, String op) {
	int r = 0;
		if ("+".equals(op)) {
		r = a + b;	
	} else if ("-".equals(op)) {
		r = a - b;
	} else if ("*".equals(op)) {
		r = a * b;
	} else if ("/".equals(op)) {
		r = a / b;
	}
	return Integer.toString(r);
}
%>
```


#### 표현식 태그

표현식(Expressions)태그는 문자열을 출력할 때 사용한다. `<%= %>`안에는 결과를 반환하는 자바코드가 와야한다. 표현식도 스크립트릿과 같이 `_jspService()`안에 순서대로 복사한다. 

```html
<body>
<h2>JSP 계산기</h2>
<form action="Calculator.jsp" method="get">
	<input type="text" name="v1" size="4" value="<%=v1%>"> 
	<select name="op">
		<option value="+" <%=selected[0]%>>+</option>
		<option value="-" <%=selected[1]%>>-</option>
		<option value="*" <%=selected[2]%>>*</option>
		<option value="/" <%=selected[3]%>>/</option>
	</select> 
	<input type="text" name="v2" size="4" value="<%=v2%>"> 
	<input type="submit" value="=">
	<input type="text" size="8" value="<%=result%>"><br>
</form> 
</body>
```

표현식 태그에 작성한 자바코드가 서블릿 파일안에서는 `out.print()`출력의 인자값(argument)으로 복사된다. 출력문을 만들 때 `out.print()`만 사용하는 것은 아니다. `out.write()`도 가능하다 즉 JSP엔진에 따라 만들어지는 자바코드가 달라 질 수 있다. 톰켓에서는 JSP 표현식에 대해 `out.print()`메서드를 만든다.


# 2. 뷰(View)


### Value Object 


데이터베이스에서 가져온 정보를 JSP페이지에 전달하려면 그 정보를 담을 객체가 필요하다. 이렇게 값을 담는 용도로 사용하는 객체를 `값 객체(value object)`라고한다. VO는 계층간 또는 객체 간에 데이터를 전달하는 데 용이하므로 `데이터 수송 객체(Data Transfer Object)`라고도 한다. 
보통 데이터베이스 테이블에 대응하여 값 객체를 정의한다.

```java
package spms.vo;

import java.util.Date;

public class Member {
	protected int 		no;
	protected String 	name;
	protected String 	email;
	protected String 	password;
	protected Date		createdDate;
	protected Date		modifiedDate;
	
	public int getNo() {
		return no;
	}
	public Member setNo(int no) {
		this.no = no;
		return this;
	}
	public String getName() {
		return name;
	}
	public Member setName(String name) {
		this.name = name;
		return this;
	}
	public String getEmail() {
		return email;
	}
	public Member setEmail(String email) {
		this.email = email;
		return this;
	}
}
```

`setXXX()`메소드의 return값을 this로 하는 이유는 다음과 같다.

```java

	//return this를 사용한 경우
	while(rs.next()) {
		members.add(new Member()
			.setNo(rs.getInt("MNO"))
			.setName(rs.getString("MNAME"))
	}

	//set메서드 반환값이 없는 경우
	Member member = new Member();
	member.setNo(rs.getInt("MNO"));
	member.setName(rs.getString("MNAME"));
	member.setEmail(rs.getString("EMAIL"));
```

객체를 생성한 즉시, setter메서드를 연속으로 호출하여 값을 할당한다. 이런 코딩 방식은 스크립팅 언어에서 널리 사용되는 방식이다. 

### RequestDispatcher, forward, include

화면생성을 위해 JSP로 작업을 위임해야한다. 이렇게 다른 서블릿이나 JSP로 작업을 위임할 때 사용하는 객체가 `RequestDispatcher`이다. 이 객체는 `HttpservletRequest`를 통해서 얻을 수 있다.

```java
RequestDispatcher rd = request.getRequestDispatcher(
			"/member/MemberList.jsp");
```

예)

```java
// request에 회원 목록 데이터 보관한다.
request.setAttribute("members", members);
			
// JSP로 출력을 위임한다.
RequestDispatcher rd = request.getRequestDispatcher(
		"/member/MemberList.jsp");
rd.include(request, response);
```

```html
<%@page import="spms.vo.Member"%>
<%@page import="java.util.ArrayList"%>
<%@ page 
	language="java" 
	contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" 
	"http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>회원 목록</title>
</head>
<body>
<h1>회원목록</h1>
<p><a href='add'>신규 회원</a></p>
<%
ArrayList<Member> members = (ArrayList<Member>)request.getAttribute(
								"members");
for(Member member : members) {
%>
<%=member.getNo()%>,
<a href='update?no=<%=member.getNo()%>'><%=member.getName()%></a>,
<%=member.getEmail()%>,
<%=member.getCreatedDate()%>
<a href='delete?no=<%=member.getNo()%>'>[삭제]</a><br>
<%} %>
</body>
</html>
```

`page` 지시자로 `import`를 처리한다. 그리고 `setAttribute`로 넘겨준 값을 `getAttribute()`를 호출하여 사용한다. 


#### forward


포워드 방식은 작업을 한 번 위임하면 다시 이전 서블릿으로 제어권이 돌아오지 않는다.
포워딩은 예외가 발생하면 화면 내용보다 좀 더 부드러운 안내 문구를 출력하는데 보통 사용한다.


* 포워딩(Servlet에서 사용하는 경우)

```java
RequestDispatcher rd = request.getRequestDispatcher("/error.jsp");
rd.forward(request, response);
```


* 포워딩(JSP에서 사용하는 경우)


```java
<jsp:forward page="/header.jsp"/>
```


#### include


인클루드 방식은 다른 서블릿으로 작업을 위임한 후 , 그 서블릿의 실행이 끝나면 다시 이전 서블릿으로 제어권이 넘어온다. 한 jsp에 여러 페이지를 포함할 때 사용한다. 

* 인클루딩(Servlet에서 사용하는 경우)


```java
RequestDispatcher rd = request.getRequestDispatcher("/Header.jsp");
rd.include(request, response);
```


* 인클루딩(JSP에서 사용하는 경우)


```java
<jsp:include page="/header.jsp"/>
```



### Reference
* [열혈강의 자바 웹 개발 워크북](http://www.yes24.com/24/Goods/13159413?Acode=101)
* [Wonjune-github](https://github.com/Wonjune/JDBCServlet/wiki/포워딩(forwarding)과-인클루딩(including))
