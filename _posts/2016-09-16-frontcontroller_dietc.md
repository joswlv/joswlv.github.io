---
layout: post
title: 프런트 컨트롤러 패턴과 의존성주입
date: 2016-09-15 20:10
categories: web
---


# 1. 프런트 컨트롤러 패턴

MVC구조에서 좀 더 유지보수가 쉬운 구조로 개선 한 것으로 두개의 컨트롤러를 사용하여 웹 브라우저 요청을 처리한다. 프론트 컨트롤러는 VO 객체의 준비, 뷰 컴포넌트의 위임, 오류 처리 등과 같은 공동 작업을 담당하고, 페이지 컨트롤러는 이름 그대로 요청한 페이지만을 위한 작업을 수행한다.


![]({{ site.url }}/images/frontcontroller.png)

1. 웹 브라우저는 회원 목록 페이지를 요청한다.
2. 프런트 컨트롤러 `DispatcherServlet`은 회원 목록 요청 처리를 담당하는 페이지 컨트롤러를 호출한다. 이때 데이터를 주고받을 바구니 역할을 할 Map 객체를 넘긴다.
3. 페이지 컨트롤러 `MemverListController`는 Dao에게 회원 목록 데이터를 요청한다.
4. MemberDao는 데이터베이스로 부터 회원 목록 데이터를 가져와서, Member객체에 담아 반환한다.
5. 페이지 컨트롤러는 Dao가 반환한 회원 목록 데이터를 Map 객체에 저장한다. 그리고 프런트 컨트롤러에게 뷰 URL을 반환한다.
6. 프런트 컨트롤러는 Map객체에 저장된 페이지 컨트롤러의 작업 결과물을 JSP가 사용할 수 있도록 ServletRequest로 옮긴다.


프런트 컨트롤러가 페이지 컨트롤러에게 작업을 위임할때 `execute()`메서드를 호출한다. 이렇게 호출하기 위해서 페이지 컨트롤러 클래스를 대표하는 Interface를 만들고 `execute()`메서드를 상속받게 만든다. 그러면 일관성있는 사용을 보장할 수 있다.

페이지 컨트롤러를 서블릿이 아닌 자바파일로 만들었기 때문에, Dao에서 리턴받은 결과값을 Map을 사용하여 데이터 저장하여, 프런트 컨트롤러로 전달한다. 프런트 컨트롤러에서는 jsp에서 이 데이터들을 사용하기 위해 서블릿 보관소에 저장한다. 이런 과정을 프런트 컨트롤러 패턴이라고 한다. <br/>


# 2. DI(Dependency Injection)을 이용한 의존성 과리

의존 객체를 사용하는 쪽과 의존 객체(또는 보관소) 사이의 결합도가 높아져서 의존 객체나 보관소에 변경이 발생하면 바로 영향을 받는다. 그리고 의존 객체를 다른 객체로 대체하기 가 어렵다. 이런 문제를 해결하기 위해 의존 객체를 외부에서 주입받는 방식이 등장하였다.

이의존객체를 전문으로 관리하는 `빈컨테이너(Java Beans Container)`를 사용한다. 빈컨테이너는 객체가 실행되기 전에 그 객체가 필요로 하는 의존 객체를 주입해주는 역할을 수행한다. 이런방식으로 의존 객체를 관리하는 것을 `의존성주입(DI:Dependency Injection)` 좀 더 일반적으로 `역제어(IoC:Inversion of Control)`라고 한다. 즉 역제어 방식의 한 예가 의존성 주입이다.

하지만 이 방법으로는 `DispatcherServlet`의 경우 페이지 컨트롤러가 추가 될 때마다 조건문을 변경해야하고, `ContextLoaderListener`도 Dao나 페이지 컨트롤러가 추가될 때마다 변경해야하는 문제가 있다.
<br/>

```java
// 의존 객체 주입을 위해 인스턴스 변수와 셋터 메서드 추가
//- 또한 의존 객체를 꺼내는 기존 코드 변경
public class MemberListController implements Controller {
  MemberDao memberDao;
  
  public MemberListController setMemberDao(MemberDao memberDao) {
    this.memberDao = memberDao;
    return this;
  }

  @Override
  public String execute(Map<String, Object> model) throws Exception {
    model.put("members", memberDao.selectList());
    return "/member/MemberList.jsp";
  }
}
```
```java
// ServletContext에 보관된 페이지 컨트롤러를 사용
@SuppressWarnings("serial")
@WebServlet("*.do")
public class DispatcherServlet extends HttpServlet {
  @Override
  protected void service(
      HttpServletRequest request, HttpServletResponse response)
      throws ServletException, IOException {
    response.setContentType("text/html; charset=UTF-8");
    String servletPath = request.getServletPath();
    try {
      ServletContext sc = this.getServletContext();
      
      // 페이지 컨트롤러에게 전달할 Map 객체를 준비한다. 
      HashMap<String,Object> model = new HashMap<String,Object>();
      model.put("session", request.getSession());
      
      Controller pageController = (Controller) sc.getAttribute(servletPath);
      
      if ("/member/add.do".equals(servletPath)) {
        if (request.getParameter("email") != null) {
          model.put("member", new Member()
            .setEmail(request.getParameter("email"))
            .setPassword(request.getParameter("password"))
            .setName(request.getParameter("name")));
        }
      } else if ("/member/update.do".equals(servletPath)) {
        if (request.getParameter("email") != null) {
          model.put("member", new Member()
            .setNo(Integer.parseInt(request.getParameter("no")))
            .setEmail(request.getParameter("email"))
            .setName(request.getParameter("name")));
        } else {
          model.put("no", new Integer(request.getParameter("no")));
        }
      } else if ("/member/delete.do".equals(servletPath)) {
        model.put("no", new Integer(request.getParameter("no")));
      } else if ("/auth/login.do".equals(servletPath)) {
        if (request.getParameter("email") != null) {
          model.put("loginInfo", new Member()
            .setEmail(request.getParameter("email"))
            .setPassword(request.getParameter("password")));
        }
      }
      
      // 페이지 컨트롤러를 실행한다.
      String viewUrl = pageController.execute(model);
      
      // Map 객체에 저장된 값을 ServletRequest에 복사한다.
      for (String key : model.keySet()) {
        request.setAttribute(key, model.get(key));
      }
      
      if (viewUrl.startsWith("redirect:")) {
        response.sendRedirect(viewUrl.substring(9));
        return;
      } else {
        RequestDispatcher rd = request.getRequestDispatcher(viewUrl);
        rd.include(request, response);
      }
      
    } catch (Exception e) {
      e.printStackTrace();
      request.setAttribute("error", e);
      RequestDispatcher rd = request.getRequestDispatcher("/Error.jsp");
      rd.forward(request, response);
    }
  }
}
```

```java
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
      
      sc.setAttribute("/auth/login.do", 
          new LogInController().setMemberDao(memberDao));
      sc.setAttribute("/auth/logout.do", new LogOutController());
      sc.setAttribute("/member/list.do", 
          new MemberListController().setMemberDao(memberDao));
      sc.setAttribute("/member/add.do", 
          new MemberAddController().setMemberDao(memberDao));
      sc.setAttribute("/member/update.do", 
          new MemberUpdateController().setMemberDao(memberDao));
      sc.setAttribute("/member/delete.do", 
          new MemberDeleteController().setMemberDao(memberDao));
      
    } catch(Throwable e) {
      e.printStackTrace();
    }
  }

  @Override
  public void contextDestroyed(ServletContextEvent event) {}
}
```
<br/>


### Reference
* [열혈강의 자바 웹 개발 워크북](http://www.yes24.com/24/Goods/13159413?Acode=101)



