---
layout: post
title: Servlet_programming
date: 2016-09-09 22:40:18
categories: web
---

# 1.CGI프로그램과 서블릿
## 1)CGI란?
![]({{ site.url }}/images/cgi.png)

위의 그림은 웹어플리케이션의 실행과정이다. 웹브라우저가 웹서버에게 요청을하고, 서버는 클라이언트가 요청한 프로그램을 찾아서 실행한다. 그리고 해당 프로그램은 작업을 수행한 후 그 결과를 웹서버에게 돌려준다. 그러면 웹서버는 그 결과를 HTTP 형식에 맞추어 웹브라우저에게 보낸다.

이대 웹서버와 프로그램 사이에 데이터를 주고받는 규칙을 `CGI(Common Gateway Interface)`라고 한다. 그래서 CGI규칙에 따라서 웹서버와 데이터를 주고 받도록 작성된 프로그램을 `CGI프로그램`이라한다.

CGI프로그램은 C나 C++, java와 같은 컴파일 언어와 스크립언어로 작성할 수 있다. 

## 2)서블릿

자바로 만든 CGI프로그램을 `서블릿(Servlet)`이라고 한다. 자바 서블릿이 다른 CGI 프로그램과 다른 점은, 웹 서버와 직접 데이터를 주고 받지 않으며, 전문 프로그램에 의해 관리된다는 점이다.

### 2-1)서블릿 컨테이너

![]({{ site.url }}/images/servlet.png)
서블릿의 생성과 실행, 소멸 등 생명주기를 관리하는 프로그램을 `서블릿 컨테이너(Servlet Container)`라고 한다. 서블릿 컨테이너가 서블릿을 대신하여 CGI규칙에 따라 웹서버와 데이터를 주고 받는다. 더 이상 CGI규칙에 대해 알 필요가 없지만, 서블릿 컨테이너와 서블릿 사이의 규칙을 알아야한다.

# 2. javax.servlet.Servlet 인터페이스

## 1)서블릿 인터페이스
서블릿 클래스는 반드시 javax.servlet.Servlet 인터페이스를 구현해야한다. 서블릿 컨테이너가 서블릿에 대해 호출할 메서드를 정의한 것이 Servlet인터페이스 이다.

>서블릿 생명주기와 관련된 메소드
>
>- init()
- service()
- destroy()

- getServletInfo()
- getServletConfig()


여기서 `service()`는 클라이언트가 요청할 때 마다 호출되는 메서드이다. 즉 실질적으로 서비스 작업을 수행하는 메서드 이다. 이 메서드에 서블릿이 해야 할 일을 작성하면된다.

`web.xml`파일을 `배치기술서(Deployment Descriptor)` 또는 `DD파일`이라고 한다. 여기에 웹어플리케이션의 배치 정보를 담고 있다. DD파일에 등록되지 않은 서블릿은 서블릿 컨테이너가 찾을 수 없다. URL과 서블릿도 매핑한다.

>*참고 페키지명+클래스명 = Fully qualified name = QName 이라한다.

## 2) 서블릿 구동 절차

1. 클라이언트의 요청이 들어오면 서블릿 컨테이너는 서블릿을 찾는다.
2. 서블릿 클래스를 로딩하고 인스턴스를 준비한 후 생성자를 호출한다. 그리고 서블릿 초기화 메서드인 init()를 호출합니다.
3. 클라이언트 요청을 처리하는 service() 메서드를 호출한다.
4. service() 메서드에서 만든 결과를 HTTP 프로토콜에 맞추어 클라이언트에 응답하는 것으로 요청처리를 완료합니다.
5. 서블릿 컨테이너를 종료하거나, 웹애플리케이션을 종료를 한다면,
6. 서블릿 컨테이너는 종료되기 전에 서블릿이 마무리 작업을 수행할 수 있도록 생성된 모든 서블릿에 대해 destroy()메서드를 호출한다.

## 3) Welcom Files

`웰컴 파일(Welcome Files)`이란, 디렉토리의 기본 웹페이지를 의미한다. 웰컴파일의 이름은 `web.xml`의 `<welcome-file-list>` 태그를 사용하여 설정할 수 있다. 이 태그의 하위 태그로 `<welcome-file>`이 있는데, 여기에 파일이름을 적으면 된다. 여러 개의 웰컴파일을 등록하게 되면, 디렉토리에서 웰컴 파일을 찾을 때 위에서부터 아래로 순차적으로 조회한다.


# 3. GenericServlet

`GenericServlet`은 추상 클래스로 서블릿 클래스가 필요로 하는 `init()`, `destroy()`, `getServletConfig()`, `getServletInfo()`를 미리 구현하여 상속한다. 즉 `service()`메소드만 구현하면 된다.

## 1) ServletRequest의 주요 메소드

메소드 | 설명
-------|------------
getRemoteAddr()|서비스를 요청한 클라이언트의 Ip주소를 반환
getScheme()|클라이언트가 요청한 URI형식 Scheme을 반환 즉 url에서 ':'문자전에 오는 값을 반환
getProtocol()|요청한 프로토콜의 이름과 버전을 반환
getParameterNames()|요청정보에서 매개변수 이름만 추출하여 반환
getParameterValues()|요청정보에서 매개변수 값만 추출하여 반환
getParameterMap()|요청 정보에서 매개변수들을 Map객체에 담아서 반환
setCharacteerEncoding()|POST요청의 매개변수에 대한 문자 집합을 설정 (기본은 `ISO-8859-1`)

## 2) servletResponse 메소드
- setContentType() : 출력할 데이터의 인코딩 형식과 문자 집합을 설정
- setCharacterEncoding() : 출력할 데이터의 문자 집합을 설정

ex) `res.setContentType("text/plain;chartest=UTF-8");

>`setContentType()`의 인자 형식이 올바르지 않으면 웹브라우저는 서버에서 받은 결과를 화면에 출력하는 대신 파일저장 대화창을 띄운다.

# 4. @WebServlet 애노테이션을 이용한 서블릿 배치 정보 설정

`Servlet 3.0` 사양부터는 애노테이션으로 서블릿 배치 정보를 설정할 수 있다. `web.xml`대신 애노테이션을 이용해 배치 정보를 작성해보자

```java
@WebServlet(
  urlPatterns={"/member/update"},
  initParams={
	  @WebInitParam(name="driver",value="com.mysql.jdbc.Driver"),
	  @WebInitParam(name="url",value="jdbc:mysql://localhost/studydb"),
	  @WebInitParam(name="username",value="study"),
	  @WebInitParam(name="password",value="study")
  }
)
public class CalculatorServlet extends GenericServlet {
	@Override
	public void service(ServletRequest req, ServletResponse res) throws ServletException, 
		....
	}
```

### Reference
- [열혈강의 자바 웹 개발 워크북](http://www.yes24.com/24/Goods/13159413?Acode=101)
