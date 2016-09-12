---
layout: post
title: 데이터 보관소
date: 2016-09-13 01:10
categories: web
---


# 1. 서블릿의 데이터 보관소

![]({{ site.url }}/images/servlet_storage.png)


### ServletContext 보관소


웹 애플리케이션이 시작 될 때 생성되어 웹 애플리케이션 종료될 때까지 유지된다. 이 보관소에 데이터를 보관하면 웹 애플리케이션이 실행되는 동안에는 모든 서블릿이 사용 할 수 있다. `application`변수를 통해 이 보관소를 참조할 수 있다.

```java
@Override
	public void init(ServletConfig config) throws ServletException {
		System.out.println("AppInitServlet 준비…");
		super.init(config);
		try {
			ServletContext sc = this.getServletContext();
			Class.forName(sc.getInitParameter("driver"));
			Connection conn = DriverManager.getConnection(
						sc.getInitParameter("url"),
						sc.getInitParameter("username"),
						sc.getInitParameter("password"));
			
			sc.setAttribute("conn", conn);
		} catch(Throwable e) {
			throw new ServletException(e);
		}
	}
	
	@Override
	public void destroy() {
		System.out.println("AppInitServlet 마무리...");
		super.destroy();
		Connection conn = 
				(Connection)this.getServletContext().getAttribute("conn"); 
		try {
			if (conn != null && conn.isClosed() == false) {
				conn.close();
			}
		} catch (Exception e) {}
		
	}
```


AppInitServlet클래스를 만들어 init()과 destroy()에서 DB커넥션과 릴리즈를 할 수 있다. 이 때 클래스의 맵핑정보가 없는데, 이를 DD파일에서 다음과 같이 입력한다.


```java
<servlet>
    <servlet-name>AppInitServlet</servlet-name>
    <servlet-class>spms.servlets.AppInitServlet</servlet-class>
    <load-on-startup>1</load-on-startup>
</servlet>
```


`<load-on-startup> </load-on-startup>`태그를 사용해 클라이언트 요청이 없어도 해당 서블릿이 웹어플리케이션이 시작 될 때 자동으로 생성된다. 이 태그의 값은 생성 순서를 의미한다.


### HttpSession 보관소

클라이언트의 최초 요청 시 생성되어 브라우저를 닫을 때까지 유지된다. 보통 로그인 할 때 이 보관소를 초기화하고, 로그아웃하면 이 보관소에 저장된 값들을 비운다. 이 보관소에 값을 보관하면 서블릿이나 JSP 페이지에 상관없이 로그아웃 하기 전까지 계속 값을 유지 가능하다. `session`변수를 통해서 이 보관소에 참조할 수 있다.


```java
// HttpSession변수에 값을 저장한다.
HttpSession session = request.getSession();
session.setAttribute("member", member);

// HttpSession변수에 저장된 값을 가져다 사용한다.
Member member = (Member)session.getAttribute("member");

// HttpSession 객체를 제거한다.
session.invalidate();
```

### ServletRequest보관소

클라이언트의 요청이 들어올 때 생성되어, 클라이언트에게 응답 할 때까지 유지된다. 이 보관소는 포워딩이나 인클루딩하는 서블릿들 사이에서 값을 공유 할 때 유용하다. `request`변수를 통해 이 보관소에 참조할 수 있다.


```java
// JSP로 출력을 위임한다.
RequestDispatcher rd = request.getRequestDispatcher("/member/MemberList.jsp");
rd.include(request, response);
```


### JspContext보관소

JSP 페이지를 실행하는 동안만 유지. 실제로 잘 쓸 일이 없다. `pageContext`변수를 통해서 이 보관소를 참조할 수 있다.



### Reference
* [열혈강의 자바 웹 개발 워크북](http://www.yes24.com/24/Goods/13159413?Acode=101)
* [slenderankle_블로그](http://slenderankle.tistory.com/170)
