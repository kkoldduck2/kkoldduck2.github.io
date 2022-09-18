---
layout: single
title:  "Spring Security "
categories: spring
tag: [web, spring, security]
toc: true
author_profile: true
---
## Spring Security 아키텍처

<img width="959" alt="image" src="https://user-images.githubusercontent.com/47748246/190884625-1a5d2513-628f-43bc-adc1-db7a3deb7f8e.png">


- 덧) AuthenticationProvider에서는 Authentication 객체에서 userId를 꺼내서 CustomUserDetailsService의 loadUserByUsername(userId)를 실행한다. 그리고 UserDetails 객체를 반환받는다.
- AuthenticationProvider는 UserDetailsService를 통해 조회한 정보와 입력받은 비밀번호가 일치하는지 확인하여, 일치한다면 인증된 AuthenticationToken을 생성하여 반환해줘야한다.
- DB에 저장된 비밀번호는 암호화되어있기 때문에, 입력으로부터 드러온 비밀번호를 PasswordEncoder를 통해 암호화하여 DB에서 조회한 사용자의 비밀번호와 매칭되는지 확인해 줘야한다.  만약 비밀번호가 매칭되지 않는 경우에는 BadCredentialsException을 발생시켜 처리해준다.

- 인증된 토큰을 AuthenticationFilter에게 전달하고, Filter에서는 LoginSuccessHandler로 전달한다.
- LoginSuccessHandler로 넘어온 Authentication 객체를 SecurityContextHolder에 저장하면 인증 과정이 끝나게 된다 .

### 1) Authentication

- 현재 접근하는 주체의 정보와 권한을 담는 인터페이스.
- Authentication 객체는 Security Context에 저장되며, SecurityContextHolder를 통해 SecurityContext에 접근하고, SecurityContext를 통해 Authentication에 접근할 수 있다.

```java
public interface Authentication extends Principal, Serializable {
	// 현재 사용자의 권한 목록을 가져옴 
	Collection<? extends GrantedAuthority> getAuthorities(); 

	// credentials(주로 비밀번호)을 가져옴 
	Object getCredentials(); Object getDetails(); 

	// Principal 객체를 가져옴. 
	Object getPrincipal(); 

	// 인증 여부를 가져옴 
	boolean isAuthenticated(); 

	// 인증 여부를 설정함 
	void setAuthenticated(boolean isAuthenticated) throws IllegalArgumentException; 
}

```

### 2) **UsernamePasswordAuthenticationToken**

- Authentication을 implements한 AbstractAuthenticationToken의 하위 클래스
    - 대충 Authentication 구현 객체임.
    - 이게 위 그림을 돌아다니면서 인증 과정을 거침
- User의 ID가 Principal 역할을 하고, Password가 Credential의 역할을 한다.
- 첫 번째 생성자 : 인증 전의 객체를 생성
- 두 번째 생성자 : 인증이 완료된 객체를 생성

```java
public class UsernamePasswordAuthenticationToken extends AbstractAuthenticationToken {
 // 주로 사용자의 ID에 해당함 
	private final Object principal; 
	// 주로 사용자의 PW에 해당함 
	private Object credentials; 

	// 인증 완료 전의 객체 생성 
	public UsernamePasswordAuthenticationToken(Object principal, Object credentials) { 
		super(null); 
		this.principal = principal; 
		this.credentials = credentials; 
		setAuthenticated(false); 
	} 

	// 인증 완료 후의 객체 생성 
	public UsernamePasswordAuthenticationToken(
									Object principal, Object credentials, 
									Collection<? extends GrantedAuthority> authorities) {
		super(authorities); 
		this.principal = principal; 
		this.credentials = credentials; 
		super.setAuthenticated(true); // must use super, as we override 
	} 
} 

public abstract class AbstractAuthenticationToken implements Authentication, CredentialsContainer { }

```

### 3) AuthenticationProvider

- 실제 인증에 대한 부분을 처리
- 인증 전의 Authentication 객체를 받아서 인증이 완료된 객체를 반환하는 역할을 한다.
- 아래와 같이 AuthenticationProvider 인터페이스를 구현해서 Custom한 AuthenticationProvider를 작성하여 AuthenticationManager에 등록하면 된다.

```java
public interface AuthenticationProvider {
 // 인증 전의 Authenticaion 객체를 받아서 인증된 Authentication 객체를 반환 
	Authentication authenticate(Authentication var1) throws AuthenticationException;
	boolean supports(Class<?> var1); 
}
```

### 4) Authentication Manager

- 인증은 AuthenticationManager에 등록된 AuthenticationProvider에 의해 처리된다 .
- 인증이 성공하면 2번째 생성자를 이용해 인증이 성공한 (isAuthenticated=true) 객체를 생성하여 Security Context에 저장한다.
- 그리고 인증 상태를 유지하기 위해 세션에 보관하며, 인증이 실패한 경우에는 AuthenticationException을 발생시킨다.

```java
public interface AuthenticationManager {
 Authentication authenticate(Authentication authentication) throws AuthenticationException; 
}
```

### 5) ProviderManager

- AuthenticationManager를 implements한 ProviderManager는 실제 인증 과정에 대한 로직을 가지고 있는 AuthenticationProvider를 List로 가지고 있으며, ProviderManager는 for문을 통해 모든 provider를 조회하면서 authenticate 처리를 한다.

```java
public class ProviderManager implements AuthenticationManager, MessageSourceAware, InitializingBean {
	public List<AuthenticationProvider> getProviders() {
		return providers; 
	} 
	public Authentication authenticate(Authentication authentication) throws AuthenticationException {
		Class<? extends Authentication> toTest = authentication.getClass(); 
		AuthenticationException lastException = null; 
		Authentication result = null; 
		boolean debug = logger.isDebugEnabled(); 
		//for문으로 모든 provider를 순회하여 처리하고 result가 나올 때까지 반복한다. 
		for (AuthenticationProvider provider : getProviders()) {
			.... 
			try {
				result = provider.authenticate(authentication);
				if (result != null) {
					copyDetails(authentication, result); 
					break; 
				} 
				} catch (AccountStatusException e) {
				prepareException(e, authentication); 
				// SEC-546: Avoid polling additional providers if auth failure is due to 
				// invalid account status throw e; 
			} 
			.... 
		} 
		throw lastException; 
	} 
}

```

### 6) CustomAuthenticationProvider 등록 방법

- ProviderManager에 우리가 직접 구현한 CustomAuthenticationProvider를 등록하는 방법은 WebSecurityConfigurerAdapter를 상속해 만든 SecurityConfig에서 할 수 있다.
- WebSecurityConfigurerAdapter의 상위 클래스에서는 AuthenticationManager를 갖고 있기 때문에 우리가 직접 만든 CustomAuthenticationProvider를 등록할 수 있다.

```java
@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
	@Bean 
	public AuthenticationManager getAuthenticationManager() throws Exception {
		return super.authenticationManagerBean(); 
	} 

	@Bean 
	public CustomAuthenticationProvider customAuthenticationProvider() throws Exception {
		return new CustomAuthenticationProvider(); 
	} 
	
	@Override 
	protected void configure(AuthenticationManagerBuilder auth) throws Exception { 
		auth.authenticationProvider(customAuthenticationProvider()); 
	}
}
```

### 7) UserDetails

- 인증에 성공하여 생성된 UserDetails 객체는 Authentication 객체를 구현한 UsernamePasswordAuthenticationToken을 생성하기 위해 사용된다 .

### 8) UserDetailsService

- UserDetailsService 인터페이스는 UserDetails 객체를 반환하는 단 하나의 메소드를 가지고 있는데, 일반적으로 이를 구현한 클래스의 내부에 UserRepository를 주입받아 DB와 연결하여 처리한다.

```java
public interface UserDetailsService {
	UserDetails loadUserByUsername(String var1) throws UsernameNotFoundException; 
}
```

```java
@RequiredArgsConstructor
@Service
public class CustomUserDetailsService implements UserDetailsService {
	private final TestService testService;

	@Override
	public UserDetails loadUserByUsername(String userId) throws UsernameNotFoundException {
		
		// userId를 이용해서 DB에서 유저 정보 가져오기 
		UserInfoVo userInfoVo = testService.getUserInfo(userId);
		
		// 없으면 exception 호출
		if(userInfoVo == null) {
			throw new UsernameNotFoundException(userId);
		}

		// 가져온 유저 정보에서 authority 꺼내 담기 
		Collection<GrantedAuthority> authorities = new ArrayList<GrantedAuthority>();
		authorities.add(new SimpleGrantedAuthority(userInfoVo.getAuthority()));
		
		return new User(userId, userInfoVo.getPassword(), authorities);
	}
}
```

### 9) Password Encoding

- AuthenticationManagerBuilder.userDetailsService().passwordEncoder()를 통해 패스워드 암호화에 사용될 PasswordEncoder 구현체를 지정할 수 있다.

```java

@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
	@Autowired
	private CustomUserDetailsService customUserDetailsService;

	@Override 
	protected void configure(AuthenticationManagerBuilder auth) throws Exception {
		auth
			.userDetailsService(customUserDetailsService)
			.passwordEncoder(passwordEncoder()); 
	} 
	
	@Bean 
	public PasswordEncoder passwordEncoder(){
		return new BCryptPasswordEncoder(); 
	}
}
```

## 구현 로직

### Spring Security 기본 설정

```java
@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http
			.authorizeRequests()
				.antMatchers("/", "/home").permitAll()  // 이 url에 대해서는 모두 허용
				.anyRequest().authenticated()   // 그 외에는 authenticated 되어야함
				.and()
			.formLogin()
				.loginPage("/login")  // 로그인 페이지를 제공하는 url 설정
				.permitAll()   // 로그인 페이지는 모두 허용
				.and()
			.logout()
				.permitAll();
	}

// 사용자 정보 in memory로 저장
	@Bean
	@Override
	public UserDetailsService userDetailsService() {
		UserDetails user =
			 User.withDefaultPasswordEncoder()
				.username("user")
				.password("password")
				.roles("USER")
				.build();

		return new InMemoryUserDetailsManager(user);
	}

// 직접 커스텀한 UserDetailService 사용
	@Override
	protected void configure(AuthenticationManagerBuilder auth) throws Exception{
		auth
		.userDetailsService(customUserDetailsService) // User : id, pw, authorities
		.passwordEncoder(passwordEncoder());
	}

}
```

### 로그인 페이지 생성

```html
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" xmlns:th="https://www.thymeleaf.org"
      xmlns:sec="https://www.thymeleaf.org/thymeleaf-extras-springsecurity3">
    <head>
        <title>Spring Security Example </title>
    </head>
    <body>
        <div th:if="${param.error}">
            Invalid username and password.
        </div>
        <div th:if="${param.logout}">
            You have been logged out.
        </div>
        <form th:action="@{/login}" method="post">
            <div><label> User Name : <input type="text" name="username"/> </label></div>
            <div><label> Password: <input type="password" name="password"/> </label></div>
            <div><input type="submit" value="Sign In"/></div>
        </form>
    </body>
</html>
```

This Thymeleaf template presents a form that captures a username and password and posts them to `/login`. As configured, Spring Security provides a **filter** that **intercepts that request** and **authenticates the user**. 

- If the user fails to authenticate, the page is redirected to `/login?error`, and your page displays the appropriate error message.
- Upon successfully signing out, your application is sent to `/login?logout`, and your page displays the appropriate success message.

참고

[https://mangkyu.tistory.com/77](https://mangkyu.tistory.com/77)

[https://spring.io/guides/gs/securing-web/](https://spring.io/guides/gs/securing-web/)
