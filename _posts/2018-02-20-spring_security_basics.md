---
layout: post
title: Spring Security 사용기
date: 2018-02-20 13:10
categories: Spring
---


# 1. 왜 Spring Security 사용했나?

- 사내 sso연동을 위한 절차가 복잡해서
- 로그인/로그아웃/권한체크 등을 쉽게 구현하기 위해

# 2. 이야기하는 것

- `UserDetails`로 로그인/로그아웃 구현 방식에 대해(REST방식 로그인)
- `SecurityContextHolder`에대해서 이해한 것들

# 3. 이야기 시작

> **로그인 관련 MySql Table정보**


```sql
create table user ( 
	username varchar(20), 
	password varchar(500), 
	name varchar(20), 
	isAccountNonExpired boolean, 
	isAccountNonLocked boolean, 
	isCredentialsNonExpired boolean, 
	isEnabled boolean 
);

create table authority (
    username varchar(20),
    authority_name varchar(20)
);
```

#### 1. `UserDetails`를 implements한 클래스를 만든다.

```java
@Getter
@Setter
@ToString
public class CustomUserModel implements UserDetails {
	private String username;
	private String password;
	private boolean isAccountNonExpired = true;
	private boolean isAccountNonLocked = true;
	private boolean isCredentialsNonExpired = true;
	private boolean isEnabled = true;
	private Collection<? extends GrantedAuthority> authorities;
}
```

#### 2. ServiceLayer에서 `UserDetailsService`를 implements한 클래스를 만든다.

```java
@Service
public class UserManageService implements UserDetailsService {
	@Autowired
	private CustomUserDao customUserDao;

	...
	
	public CustomUserModel readUser(String userName) {
		return customUserDao.readUser(userName);
	}

	public Collection<GrantedAuthority> getAuthorities(String username) {
		return customUserDao.readAuthority(username);
	}

	@Override
	public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
		CustomUserModel customUserModel = customUserDao.readUser(username);
		customUserModel.setAuthorities(getAuthorities(username));
		return customUserModel;
	}
}
```

> 참고 Mybatis를 사용할 경우 ResultMap으로 `Collection<GrantedAuthority>` return 받음!!
>


```xml
<resultMap id="authorityMap" type="org.springframework.security.core.authority.SimpleGrantedAuthority">
	<constructor>
		<idArg column="authority_name" javaType="String"/>
	</constructor>
</resultMap>
```

#### 3. SecurityConfig 파일은 만든다.

> **REST방법을 사용하는 경우 다음과 같이 사용한다. 아이디/패스워드를 json을 받아서 처리하니.. formLogin으로 처리해도 되지만, 따로 json처리를 해줘야하는 귀찮음이 있다.**

```java
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
@ComponentScan(basePackages = {"com.addinfra.customtargeting.security"})
public class SecurityConfig extends WebSecurityConfigurerAdapter {
	@Autowired
	UserManageService userManageService;

	@Autowired
	RestUnauthorizedEntryPoint authenticationEntryPoint;

	@Autowired
	RestAccessDeniedHandler restAccessDeniedHandler;

	@Autowired
	LogoutHandler logoutHandler;

	@Autowired
	public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
		auth.userDetailsService(userManageService);
	}

	@Override
	public void configure(WebSecurity web) throws Exception {
		web.ignoring()
				.antMatchers("/*.js", "/*.css,/*.png", "/*.jpg", "/*.otf", "/*.eot", "/*.svg", "/*.ttf", "/*.woff", "/*.woff2", "/signin.html");
	}

	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http.csrf().disable();
		http.authorizeRequests()
				.antMatchers(HttpMethod.POST, "/login").permitAll()
				.antMatchers(HttpMethod.POST, "/logout").permitAll()
				.antMatchers(HttpMethod.OPTIONS, "/**").permitAll()
				.antMatchers("/confirmAdList.html").hasAnyAuthority("admin")
				.antMatchers("/userList.html").hasAnyAuthority("admin")
				.anyRequest().authenticated()
				.and()
				.exceptionHandling()
				.authenticationEntryPoint(authenticationEntryPoint)
				.accessDeniedHandler(restAccessDeniedHandler);

		http.logout()
				.invalidateHttpSession(true)
				.deleteCookies("JSESSIONID")
				.logoutSuccessHandler(logoutHandler);
	}

	@Bean
	@Override
	public AuthenticationManager authenticationManagerBean() throws Exception {
		return super.authenticationManagerBean();
	}
}
```

다양한 Handler를 사용할 수 있다. 각 상황에 맞는 Handler를 구현해서 사용하면 된다.

> 프론트를 AngularJs를 사용하여 구현했다. 이때 logoutSuccessHandler를 사용해 pageRedirct를 할려고 했지만 잘되지 않아.. 우회적으로 Response를 프론트에서 받아 pageRedirct를 했다.
> 
> 그리고 기본적으로 Spring Security에서 `/logout`url요청에 대해서 `filter`가 걸려있다. 따로 Controller에서 `/logout`을 구현하지 않아도 자동으로 Session과 SecurityContextHolder가 초기화 된다.
> 
```java
@Component
public class LogoutHandler implements LogoutSuccessHandler {
	@Override
	public void onLogoutSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
		SecurityUtils.sendResponse(response,HttpServletResponse.SC_OK,"logoutOk");
	}
}
```

#### Login Controller 구현

formLogin과 달리 login처리를 따로 해줘야한다. 

```java
@RestController
public class LoginController {
	@Autowired
	UserManageService userManageService;

	@Autowired
	AuthenticationManager authenticationManager;

	@PostMapping("/login")
	public String login(@RequestBody AuthenticationRequest authenticationRequest, HttpSession session) {
		try {
			String username = authenticationRequest.getUsername();
			String password = authenticationRequest.getPassword();

			UsernamePasswordAuthenticationToken token = new UsernamePasswordAuthenticationToken(username, password);
			Authentication authentication = authenticationManager.authenticate(token);

			SecurityContextHolder.getContext().setAuthentication(authentication);
			session.setAttribute(HttpSessionSecurityContextRepository.SPRING_SECURITY_CONTEXT_KEY,
					SecurityContextHolder.getContext());
			
			return new AuthenticationToken(authentication.getName(), userManageService.getLevel(authentication.getAuthorities()), session.getId());

		} catch (BadCredentialsException e) {
			return JSONObject.quote("check_pw");
		} catch (InternalAuthenticationServiceException e) {
			return JSONObject.quote("check_id");
		} catch (Exception e) {
			return JSONObject.quote(e.getMessage());
		}
	}
}
```

>`UsernamePasswordAuthenticationToken`에서 id/pw로 token을 만들어 `authenticationManager`에서 authenticate(인증)을 받고 `SecurityContextHolder`에 인증정보를 저장한다. 즉 authorization(인가)된 사용자라고 저장한다.

#### Logout에 대해서

Spring Scurity에 `LogoutFilter`클래스가 있다. 여기서 하는 일은 `/logout` url로 요청이 오면 `SecurityContextHolder`를 clear하고 session을 삭제한다.

```java
public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain) throws IOException, ServletException {
    HttpServletRequest request = (HttpServletRequest)req;
    HttpServletResponse response = (HttpServletResponse)res;
    if (this.requiresLogout(request, response)) {
        Authentication auth = SecurityContextHolder.getContext().getAuthentication();
        if (this.logger.isDebugEnabled()) {
            this.logger.debug("Logging out user '" + auth + "' and transferring to logout destination");
        }

        this.handler.logout(request, response, auth);
        this.logoutSuccessHandler.onLogoutSuccess(request, response, auth);
    } else {
        chain.doFilter(request, response);
    }
}
```

아무런 설정을 하지 않으면 기본적으로 생성된 ScurityContextHolder는 `ThreadLocalSecurityContextHolderStrategy`에 저장된다. 

여기서 궁금한점은 같은 아디로 여러 곳에서 로그인을 했을 때 한곳에서 로그아웃을 하면 다른 곳에서 로그인한 사용자도 로그아웃이 될까? (결론은 다른곳 사용자는 로그아웃이 안된다.)

```java
public void logout(HttpServletRequest request, HttpServletResponse response, Authentication authentication) {
    Assert.notNull(request, "HttpServletRequest required");
    if (this.invalidateHttpSession) {
        HttpSession session = request.getSession(false);
        if (session != null) {
            this.logger.debug("Invalidating session: " + session.getId());
            session.invalidate();
        }
    }

    if (this.clearAuthentication) {
        SecurityContext context = SecurityContextHolder.getContext();
        context.setAuthentication((Authentication)null);
    }

    SecurityContextHolder.clearContext();
}
```

위 소스는 혼란이 왔던 부분이다.

id/pw로만 인증을 할텐데.. 어떻게 다른 곳에서 로그인한 것을 구분할까... 여기에 답은 `AuthenticationManager`에서 사용자 session값도 함께 저장을 하기 때문에 각각에 사용자에 대해서 로그아웃처리를 할 수 있던것이다.

그럼 AuthenticationManager를 사용하지 않고 id/pw로만 인증과정을 직접구현해서 사용하면 한사람이 로그아웃하면 동일아이디로 로그인 한사용자 모두가 로그아웃 되겠지..


### Reference

* [https://www.slideshare.net/madvirus/ss-36809454](https://www.slideshare.net/madvirus/ss-36809454)
* [http://cusonar.tistory.com/17?category=607756](http://cusonar.tistory.com/17?category=607756)
* [http://devjinys.tistory.com/entry/Spring-Boot-%EA%B8%B0%EB%B0%98-Spring-Security-integration-with-REST-Controller](http://devjinys.tistory.com/entry/Spring-Boot-%EA%B8%B0%EB%B0%98-Spring-Security-integration-with-REST-Controller)