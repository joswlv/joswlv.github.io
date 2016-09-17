---
layout: post
title: 리플렉션API와 프로퍼티를 이용한 객체관리
date: 2016-09-17 20:29
categories: web
---


# 1. Reflection API

리플렉션이란 객체를 통해 클래스의 정보를 분석해 내는 프로그램 기법을 말한다. 투영, 반사 라는 사전적인 의미를 지니고 있다. 

자바는 스크립트 언어가 아닌 컴파일 언어이다. 물론 .java -> .class -> 실행이라는 2단계의 메커니즘을 가지고 있지만 컴파일 언어로 분리하는 게 옳다. 원래 자바에서는 동적으로 객체를 생성하는 기술이 없었다. 그리고 동적으로 인스턴스를 생성하는 Reflection으로 그 역활을 대신하게 된다.

즉 클래스나 메서드의 내부 구조를 들여다 볼 때 사용하는 도구라는 뜻이다.<br/>



| 메서드 | 설명 |
|---------|----------|
| Class.newInstance() | 주어진 클래스의 인스턴스를 생성 |
| Class.getName() | 클래스의 이름을 반환 |
| Class.getMethods() | 클래스에 선언된 모든 public 메서드의 목록을 배열로 반환 |
| Method.invoke() | 해당 메서드를 호출 |
| Method.getParameterTypes() | 메서드의 매개변수 목록을 배열로 반환 |


<br/>


#### VO 객체 생성을 Reflection을 사용하여 만드는 시나리오



![]({{ site.url }}/images/voobject.png)


1. 웹 브라우저는 회원 등록을 요청합니다. 사용자가 입력한 매개변수 값을 서블릿에 전달한다.
2. 프런트 컨트롤러 `DispatcherServlet`은 회원 등록을 처리하는 페이지 컨트롤러에게 어떤 데이터가 필요한지 물어본다. 페이지 컨트롤러 `MemberAddController`는 작업하는 데 필요한 데이터의 이름과 타입 정보를 담은 배열을 리턴한다.
3. 프런트 컨트롤러는 `ServletRequestDataBinder`를 이용하여, 요청 매개변수로부터 페이지 컨트롤러가 원하는 형식의 값객체(예: Member, Integer, Date등)를 만든다.
4. 프런트 컨트롤러는 `ServletRequestDataBinder`가 만들어 준 값 객체를 Map에 저장한다.
5. 프런트 컨트롤러는 페이지 컨트롤러 `MemberAddController`를 실행한다. 페이지 컨트롤러의 execute()를 호출할 때, 값이 저장된 Map 객체를 매개변수로 넘긴다.

아래 소스는 프런트 컨트롤러에서 VO 객체 생성을 Reflection을 사용하여 자동화 한것이다.



```java
// DataBinding 처리
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
      
      if (pageController instanceof DataBinding) {
        prepareRequestData(request, model, (DataBinding)pageController);
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

  private void prepareRequestData(HttpServletRequest request,
      HashMap<String, Object> model, DataBinding dataBinding)
      throws Exception {
    Object[] dataBinders = dataBinding.getDataBinders();
    String dataName = null;
    Class<?> dataType = null;
    Object dataObj = null;
    for (int i = 0; i < dataBinders.length; i+=2) {
      dataName = (String)dataBinders[i];
      dataType = (Class<?>) dataBinders[i+1];
      dataObj = ServletRequestDataBinder.bind(request, dataType, dataName);
      model.put(dataName, dataObj);
    }
  }
}
```

<br/>

```java
public class ServletRequestDataBinder {
  public static Object bind(
      ServletRequest request, Class<?> dataType, String dataName) 
      throws Exception {
    if (isPrimitiveType(dataType)) {
      return createValueObject(dataType, request.getParameter(dataName));
    }
    
    Set<String> paramNames = request.getParameterMap().keySet();
    Object dataObject = dataType.newInstance();
    Method m = null;
    
    for (String paramName : paramNames) {
      m = findSetter(dataType, paramName);
      if (m != null) {
        m.invoke(dataObject, createValueObject(m.getParameterTypes()[0], 
            request.getParameter(paramName)));
      }
    }
    return dataObject;
  }
  
  private static boolean isPrimitiveType(Class<?> type) {
    if (type.getName().equals("int") || type == Integer.class ||
        type.getName().equals("long") || type == Long.class ||
        type.getName().equals("float") || type == Float.class ||
        type.getName().equals("double") || type == Double.class ||
        type.getName().equals("boolean") || type == Boolean.class ||
        type == Date.class || type == String.class) {
      return true;
    }
    return false;
  }
  
  private static Object createValueObject(Class<?> type, String value) {
    if (type.getName().equals("int") || type == Integer.class) {
      return new Integer(value);
    } else if (type.getName().equals("float") || type == Float.class) {
      return new Float(value);
    } else if (type.getName().equals("double") || type == Double.class) {
      return new Double(value);
    } else if (type.getName().equals("long") || type == Long.class) {
      return new Long(value);
    } else if (type.getName().equals("boolean") || type == Boolean.class) {
      return new Boolean(value);
    } else if (type == Date.class) {
      return java.sql.Date.valueOf(value);
    } else {
      return value;
    }
  }
  
  private static Method findSetter(Class<?> type, String name) {
    Method[] methods = type.getMethods();
    
    String propName = null;
    for (Method m : methods) {
      if (!m.getName().startsWith("set")) continue;
      
      propName = m.getName().substring(3);
      if (propName.toLowerCase().equals(name.toLowerCase())) {
        return m;
      }
    }
    return null;
  }
}
```

```java
//VO객체
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
	
	....
	
}
```

<br/>

# 2. Property를 이용한 객체관리

DB소스값, 페이지 컨트롤러 위치를 프로퍼티 파일로 만들고 이를 읽어 객체를 생성하고, DI를 하는 클래스를 만든다. 이를 ContextLoaderListener의 contextInitialized()에서 프로퍼티 값을 얻는 get메서드를 만들고 프런트 컨트롤러에서 사용한다. 


#### 시나리오


1. 웹 어플리케이션이 시작되면 서블릿 컨테이너는 ContextLoaderListener의 contextInitialized()메서드를 호출한다.
2. contextInitialized()메서드에서는 ApplicationContext를 생성한다. 이때 생성자에 프로퍼티 파일의 경로를 매개변수로 넘겨준다.
3. ApplicationContext는 프로퍼티 파일의 내용을 읽는다.
4. 프로퍼티 파일에 선언된 대로 객체를 생성하여 객체 테이블에 저장한다.
5. 객체 테이블에 저장된 각 객체에 대해 의존 객체를 찾아서 할당 한다.

이렇게 되면 페이지 컨트롤러나 DAO를 만들 때마다 더 이상 ContextLoaderListener를 변경할 필요가 없어진다.

<br/>


```java
//application-context.properties
#1. for ApplicationContext.
jndi.dataSource=java:comp/env/jdbc/studydb
memberDao=spms.dao.MySqlMemberDao
/auth/login.do=spms.controls.LogInController
/auth/logout.do=spms.controls.LogOutController
/member/list.do=spms.controls.MemberListController
/member/add.do=spms.controls.MemberAddController
/member/update.do=spms.controls.MemberUpdateController
/member/delete.do=spms.controls.MemberDeleteController
```

```java
// 프로퍼티 파일 적용 : ApplicationContext 사용

@WebListener
public class ContextLoaderListener implements ServletContextListener {
  static ApplicationContext applicationContext;
  
  public static ApplicationContext getApplicationContext() {
    return applicationContext;
  }
  
  @Override
  public void contextInitialized(ServletContextEvent event) {
    try {
      ServletContext sc = event.getServletContext();
      
      String propertiesPath = sc.getRealPath(
          sc.getInitParameter("contextConfigLocation"));
      applicationContext = new ApplicationContext(propertiesPath);
      
    } catch(Throwable e) {
      e.printStackTrace();
    }
  }
  
  @Override
  public void contextDestroyed(ServletContextEvent event) {}
}
```

```java
package spms.context;
/*
만약 프로퍼티 키가 "jndi"로 시작한다면 객체를 생성하지 않고, InitialContext를 통해서 얻는다.
InitialContext의 lookup()메서드는 JNDI인터페이스를 통해 톰켓 서버에 등록된 객체를 찾는다.
그 밖의 객체는 Class.forName()을 호출하여 클래스를 로딩하고, newInstance()를 사용하여 인스턴스를 생성한다.

injectDpendency()는 "jndi."로 시작하는 톰켓서버에서 제공한 객체는 의존객체를 주입해서는 안되기 때문에 제외 했고 
그 외 객체들은 set으로 시작하는 메서드를 찾고 
그 메서드의 매개변수와 타입이 일치하는 객체를 objTable에서 찾아
셋터 메서드를 호출하는 방식이다.
*/

// 프로퍼티 파일을 이용한 객체 준비
public class ApplicationContext {
  Hashtable<String,Object> objTable = new Hashtable<String,Object>();
  
  public Object getBean(String key) {
    return objTable.get(key);
  }
  
  public ApplicationContext(String propertiesPath) throws Exception {
    Properties props = new Properties();
    props.load(new FileReader(propertiesPath));
    
    prepareObjects(props);
    injectDependency();
  }
  
  private void prepareObjects(Properties props) throws Exception {
    Context ctx = new InitialContext();
    String key = null;
    String value = null;
    
    for (Object item : props.keySet()) {
      key = (String)item;
      value = props.getProperty(key);
      if (key.startsWith("jndi.")) {
        objTable.put(key, ctx.lookup(value));
      } else {
        objTable.put(key, Class.forName(value).newInstance());
      }
    }
  }
  
  private void injectDependency() throws Exception {
    for (String key : objTable.keySet()) {
      if (!key.startsWith("jndi.")) {
        callSetter(objTable.get(key));
      }
    }
  }

  private void callSetter(Object obj) throws Exception {
    Object dependency = null;
    for (Method m : obj.getClass().getMethods()) {
      if (m.getName().startsWith("set")) {
        dependency = findObjectByType(m.getParameterTypes()[0]);
        if (dependency != null) {
          m.invoke(obj, dependency);
        }
      }
    }
  }
  
  private Object findObjectByType(Class<?> type) {
    for (Object obj : objTable.values()) {
      if (type.isInstance(obj)) {
        return obj;
      }
    }
    return null;
  }
}
```

```java
// 페이지 컨트롤러를 찾을 때 ApplicationContext의 사용
@WebServlet("*.do")
public class DispatcherServlet extends HttpServlet {
  @Override
  protected void service(
      HttpServletRequest request, HttpServletResponse response)
      throws ServletException, IOException {
    response.setContentType("text/html; charset=UTF-8");
    String servletPath = request.getServletPath();
    try {
      ApplicationContext ctx = ContextLoaderListener.getApplicationContext();
      
      // 페이지 컨트롤러에게 전달할 Map 객체를 준비한다. 
      HashMap<String,Object> model = new HashMap<String,Object>();
      model.put("session", request.getSession());
      
      Controller pageController = (Controller) ctx.getBean(servletPath);
      if (pageController == null) {
        throw new Exception("요청한 서비스를 찾을 수 없습니다.");
      }
      
      if (pageController instanceof DataBinding) {
        prepareRequestData(request, model, (DataBinding)pageController);
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

  private void prepareRequestData(HttpServletRequest request,
      HashMap<String, Object> model, DataBinding dataBinding)
      throws Exception {
    Object[] dataBinders = dataBinding.getDataBinders();
    String dataName = null;
    Class<?> dataType = null;
    Object dataObj = null;
    for (int i = 0; i < dataBinders.length; i+=2) {
      dataName = (String)dataBinders[i];
      dataType = (Class<?>) dataBinders[i+1];
      dataObj = ServletRequestDataBinder.bind(request, dataType, dataName);
      model.put(dataName, dataObj);
    }
  }
}
```
```xml
//DD파일
  <!-- 컨텍스트 초기화 파라미터 -->
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/application-context.properties</param-value>
  </context-param>
```


# 3. 애노테이션을 이용한 객체 관리

DAO나 페이지 컨트롤러에 애노테이션을 직접 규칙을 설정해 붙인다. 애노테이션의 기본값은 객체이름으로 보통한다. ApplicationContext에서 애노테이션이 붙은 클래스를 읽어 애노테이션 값을 key값으로 객체의 인스턴스를 값으로 objTable에 추가 해 이전과 같은 방식으로 작동한다.

>애노테이션을 정의할 때 잊지 말아야할 것은 애노테이션의 유지 정책을 지정하는 것이다. `애노테이션 유지 정책`이란 애노테이션 정보를 언제까지 유지할 것인지 설정하는 문법이다.
>>
>>
| 정책 | 설명 |
|-----|-----------|
| RetentionPolicy.SOURCE | 소스 파일에서만 유지, 컴파일할 때 제거됨, 즉 클래스 파일에 애노테이션 정보가 남아 있지 않음 |
| RetentionPolicy.CLASS | 클래스 파일에 기록됨, 실행시에는 유지되지 않음, 즉 실행 중에서는 클래스에 기록된 애노테이션 값을 꺼낼 수 없음(기본 정책) |
| RetentionPolicy.RUNTIME | 클래스 파일에 기록됨, 실행시에도 유지됨, 즉 실행 중에 클래스에 기록된 애노테이션 값을 참조할 수 있음 |


<br/>

```java
package spms.annotation;

import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;

@Retention(RetentionPolicy.RUNTIME)
public @interface Component {
  String value() default "";
}
```

```java
/*
애노테이션이 붙은 클래스를 찾기 위해 'Reflections'라는 오픈소스사용
*/

// 프로퍼티 파일 및 애노테이션을 이용한 객체 준비
public class ApplicationContext {
  Hashtable<String,Object> objTable = new Hashtable<String,Object>();
  
  public Object getBean(String key) {
    return objTable.get(key);
  }
  
  public ApplicationContext(String propertiesPath) throws Exception {
    Properties props = new Properties();
    props.load(new FileReader(propertiesPath));
    
    prepareObjects(props);
    prepareAnnotationObjects();
    injectDependency();
  }
  
  private void prepareAnnotationObjects() 
      throws Exception{
    Reflections reflector = new Reflections("");
    
    Set<Class<?>> list = reflector.getTypesAnnotatedWith(Component.class);
    String key = null;
    for(Class<?> clazz : list) {
      key = clazz.getAnnotation(Component.class).value();
      objTable.put(key, clazz.newInstance());
    }
  }
  
  ....
  
```

```java
// Annotation 적용
@Component("/member/list.do")
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

<br/>

JNDI객체나 외부 라이브러리에 들어 있는 객체는 애노테이션을 적용할 수 없기 때문에 프로퍼티 파일에 등록해서 사용한다.









### Reference
* [열혈강의 자바 웹 개발 워크북](http://www.yes24.com/24/Goods/13159413?Acode=101)
* [Minsub's Blog](http://gyrfalcon.tistory.com/entry/Java-Reflection)