---
layout: post
title: Interceptor
date: 2016-10-02 22:04
categories: Spring
---


# Interceptor 


ServletFilter와 유사하게 DispatcherServlet이 Controller를 호출하기 전/후에 요청 및 응답을 참조, 가공할 수 있는 일종의 필터 역할 

Filter와 다른점은 Filter는 DispatcherServlet보다 먼저 실행되기 때문에 Servlet관련 변수들을 사용할 수 없지만 Interceptor는 DispatcherServlet안에서 실행되기 때문에 Servlet관련 변수들을 사용할 수 있다.


### `HandlerInterceptor` interface

* preHandler : 컨트롤러 실행 전 호출
* postHandler : 컨트롤러가 정상적으로 실행된 후 호출 (예외 발생하면 호출되지 않음)
* afterCompletion : 뷰 렌더링 후 호출 (예외 발생하면 메쏘드 인자로 전달됨)



### `HandlerInterceptorAdapter` class

* `HandlerInterceptor` 인터페이스의 모든 메쏘드를 빈(empty) 구현한 클래스


## Interceptor 설정 - SpringMVC에서

Login Session을 체크하는 interceptor예를 보면 다음과 같다.

`servlet-context`에서 다음과 같이 추가를한다.

```xml
    <mvc:interceptors>
        <mvc:interceptor>
            <mvc:mapping path="/" />

            <mvc:mapping path="/member/*" />

            <mvc:exclude-mapping path="/login" />
            <mvc:exclude-mapping path="/logout" />

            <bean class="com.nhnent.springmvc.interceptor.LoginCheckInterceptor" />
        </mvc:interceptor>
    </mvc:interceptors>
```

그리고 `LoginCheckInterceptor`클래스를 작성한다.


```java
public class LoginCheckInterceptor extends HandlerInterceptorAdapter {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        HttpSession session = request.getSession();

        Member loginInfo = (Member) session.getAttribute("member");
        if (loginInfo == null) {
            throw new UnAuthorizedException();
        }

        return super.preHandle(request, response, handler);
    }
}
```

`login`과정과 `logout`과정은 interceptor 할 필요가 없기 때문에 exclude를 설정한다.

## Interceptor 설정 - Spring Boot에서

`HandlerInterceptor` 인터페이스를 사용하면 `preHandle()`, `postHandle()`와 `afterCompletion()`를 모두 오버라이딩을 해야한다. 예를 들면 아래와 같다.

```java
public class LoggingInterceptor implements HandlerInterceptor  {
	@Override
	public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
		throws Exception {
		System.out.println("---Before Method Execution---");
		return true;
	}
	@Override
	public void postHandle(	HttpServletRequest request, HttpServletResponse response,
			Object handler, ModelAndView modelAndView) throws Exception {
		System.out.println("---method executed---");
	}
	@Override
	public void afterCompletion(HttpServletRequest request, HttpServletResponse response,
			Object handler, Exception ex) throws Exception {
		System.out.println("---Request Completed---");
	}
} 
```

`HandlerInterceptorAdapter`를 상속받으면 필요한 메소드만 사용할 수 있다. 예를 들면 아래와 같다.

```java
public class TransactionInterceptor extends HandlerInterceptorAdapter {
	@Override
	public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
		throws Exception {
		System.out.println("Got request to save data : name:"+request.getParameter("name"));
		return true;
	}
} 
```

그리고 Config파일을 만들어 `addInterceptor()`메소드를 사용하여 앞서 만든 Interceptor java파일들을 추가한다. 예는 아래와 같다.

```java
@Configuration  
@EnableWebMvc   
public class AppConfig extends WebMvcConfigurerAdapter  {  
	@Override
	public void addInterceptors(InterceptorRegistry registry) {
	    registry.addInterceptor(new LoggingInterceptor());
	    registry.addInterceptor(new TransactionInterceptor()).addPathPatterns("/person/save/*");
	}
}
```

마지막으로 `WebApplicationInitializer`인터페이스를 사용하는 java파일을 만들어 `DispatcherServlet`에 앞서 설정한 `AppConfig`를 Servlet-context로 등록시킨다. 예를들면 아래와 같다.

```java
public class WebAppInitializer implements WebApplicationInitializer {
	public void onStartup(ServletContext servletContext) throws ServletException {  
        AnnotationConfigWebApplicationContext ctx = new AnnotationConfigWebApplicationContext();  
        ctx.register(AppConfig.class);  
        ctx.setServletContext(servletContext);    
        Dynamic dynamic = servletContext.addServlet("dispatcher", new DispatcherServlet(ctx));  
        dynamic.addMapping("/");  
        dynamic.setLoadOnStartup(1);  
   }  
}
```

### Reference
* [Spring MVC HandlerInterceptor Annotation Example with WebMvcConfigurerAdapter](http://www.concretepage.com/spring/spring-mvc/spring-handlerinterceptor-annotation-example-webmvcconfigureradapter)