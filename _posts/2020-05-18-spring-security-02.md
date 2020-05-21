---
layout: post
title: "Spring security를 이용한 회원가입/로그인 예제"
categories: [spring]
comments: true
tags:
  - Spring Boot
  - Spring Security
---

# Spring boot에서 Spring security 사용하기
*스프링 부트로 배우는 자바 웹 개발* 책을 참고했다. 
[zip파일로 소스코드 전체 다운로드](https://github.com/thecodinglive/JPub-JavaWebService/archive/master.zip)
(전체 파일 중 memberApp 부분에 해당한다.)

## Spring security 설정 
스프링부트에서 스프링 시큐리티 설정은 `WebSecurityConfigurerAdapter`라는 클래스를 상속받은 클래스에서 configure 메서드를 오버라이딩 해 조정할 수 있다.<br>
configure 메서드의 파라미터로는 조정하고자 하는 보안 객체가 들어간다. 

*SecurityConfig.java*
```java
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecurityConfig extends WebSecurityConfigurerAdapter {
	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http
				.authorizeRequests()
				//접근 허용
				.antMatchers("/css/**", "/js/**", "/images/**", "/resources/**", "/h2-console/**", "/webjars/**", "/signup", "/home", "/").permitAll()
				.anyRequest().authenticated()
				.and()
				//iframe 허용
				.headers().
				frameOptions().disable()
				.and()
				//DB 어드민 접속 허용
				.csrf()
				.ignoringAntMatchers("/h2-console/**")
				.and()
				//로그인
				.formLogin()
				.loginPage("/login")
				.loginProcessingUrl("/sign-in")
				.defaultSuccessUrl("/welcome")
				.failureUrl("/login?error")
				.usernameParameter("email")
				.passwordParameter("passwd")
				.permitAll()
				.and()
				//예외처리 
				.exceptionHandling().accessDeniedHandler(MemberAccessDeniedHandler())
				.and()
				.rememberMe()
				.key("jpub")
				.rememberMeParameter("remember-me")
				.rememberMeCookieName("jpubcookie")
				.tokenValiditySeconds(86400) //1day
				.tokenRepository(rememberMeTokenService()).userDetailsService(myUserService())
				.and()
				//로그아웃
				.logout()
				.invalidateHttpSession(true)
				.clearAuthentication(true)
				.logoutRequestMatcher(new AntPathRequestMatcher("/logout"))
				.logoutSuccessUrl("/login?logout")
				.permitAll();
	}


	@Override
	protected void configure(AuthenticationManagerBuilder auth) throws Exception {
		auth.userDetailsService(myUserService()).passwordEncoder(bCryptPasswordEncoder());
	}

	@Bean
	public FilterRegistrationBean getSpringSecurityFilterChainBindedToError(
			@Qualifier("springSecurityFilterChain") Filter springSecurityFilterChain) {

		FilterRegistrationBean registration = new FilterRegistrationBean();
		registration.setFilter(springSecurityFilterChain);
		registration.setDispatcherTypes(EnumSet.allOf(DispatcherType.class));
		return registration;
	}


	@Bean
	public BCryptPasswordEncoder bCryptPasswordEncoder() {
		return new BCryptPasswordEncoder();
	}


	@Bean
	public MyUserService myUserService() throws Exception {
		return new MyUserService();
	}

	@Bean
	public RememberMeTokenService rememberMeTokenService() throws Exception{
		return new RememberMeTokenService();
	}

	@Bean
	public MemberAccessDeniedHandler MemberAccessDeniedHandler() throws Exception{
		return new MemberAccessDeniedHandler();
	}
}
```

- **@EnableWebSecurity**
  - @Configuration 클래스에 @EnableWebSecurity 어노테이션을 추가하여 Spring Security 설정할 클래스라고 정의한다. 
- **WebSecurityConfigurerAdapter 클래스**
  - WebSecurityConfigurer 인스턴스를 편리하게 생성하기 위한 클래스
- **BCryptPasswordEncoder**
  - BCryptPasswordEncoder는 Spring Security에서 제공하는 비밀번호 암호화 객체이다.
    > BCryptPasswordEncoder는 BCrypt 해시 함수를 이용하여 비밀번호를 저장하는 방법을 사용
  - Service에서 비밀번호를 암호화할 수 있도록 Bean으로 등록

security 설정은 `configure()` 메서드를 오버라이딩해 추가할 수 있다.
각 설정은 and로 연결한다. 
- **configure(HttpSecurity http)**
	- HttpSecurity를 통해 http 요청에 대한 웹 기반 보안 수준을 구성할 수 있다. 
	- **authorizeRequests()**
		- HttpServletRequest에 따라 접근(access)을 제한
		- antMatchers() 메서드로 특정 경로를 지정하며, permitAll(), hasRole() 메서드로 역할(Role)에 따른 접근 설정을 정의
		- antMathers는 접근 제어를 하지 않을 경로들을 지정
			- .antMatchers("/admin/**").hasRole("ADMIN")
			/admin 으로 시작하는 경로는 ADMIN 롤을 가진 사용자만 접근 가능
			- .antMatchers("/user/myinfo").hasRole("MEMBER")
			/user/myinfo 경로는 MEMBER 롤을 가진 사용자만 접근 가능
			- .antMatchers("/**").permitAll()
			모든 경로에 대해서는 권한없이 접근 가능
			- .anyRequest().authenticated()
			모든 요청에 대해, 인증된 사용자만 접근하도록 설정
	- **csrf()**
		- ignoringAntMatchers() 메서드로 CSRF 보안을 특정 경로에 대해 비활성화
			- 예제에서는 h2-console을 사용할 수 있도록 보안 해제 
	- **formLogin()**
		- 로그인 설정
		- loginPage: 아이디, 패스워드 정보를 입력하는 화면 경로
			- authorizeRequest 메서드에서 인가되지 않은 url로 이동할 경우 입력한 로그인 페이지로 리다이렉트됨
		- loginProcessingUrl: 입력한 로그인 정보에 대해 POST 방식으로 인증을 요청하는 경로
		- usernameParameter / passwordParameter: 기본 username, password 필드 대신 다른 필드로 인증
			- html의 태그 name에 다른 속성을 정의해 사용  
	- **exceptionHandling()**
		- 인가되지 않은 사용자가 페이지에 접근할 경우 발생하는 AccessDeniedException을 403에러 페이지 대신 다른 방법으로 처리
		- accessDeniedHandler를 구현
	- **rememberMe()**
		- 로그인 유지를 원할 때 사용
		*([Remember-Me 문서]<>를 참조)*
	- **logout()**
		- 로그아웃을 지원하는 메서드
		- invalidateHttpSession: HTTP 세션을 초기화
		- logoutRequestMatcher: 기본 로그아웃 url(/logout)이 아닌 다른 url로 재정의
		- deleteCookies("KEY명"): 로그아웃 시, 특정 쿠기를 제거하고 싶을 때 사용하는 메서드

	다음은 가장 기본적인 형태의 HttpSecurity 설정이다.
	```java
	@Override
	protected void configure(HttpSecurity http) throws Exception {
			http.authorizeRequests().antMatchers("/**").hasRole("USER").and().formLogin();
	}
	```

>authenticationManager의 구현체는 ProviderManager이다. 인증구현은 AuthentiicationProvider가 맡는다. 
## Validation 설정 
`MemberValidator`에서 회원가입 입력값을 검증한다. 
form으로 회원가입 정보를 제출하면 controller에서 다음과 같이 압력값 검증을 요청한다. 
```java
memberValidator.validate(userForm, bindingResult);

if (bindingResult.hasErrors()) {
  log.debug("valid error");
  return "signup";
}
```

아래는 MemberValidator의 validate 메서드이다. 
```java
@Override
public void validate(Object target, Errors errors) {
  Member member = (Member)target;

  ValidationUtils.rejectIfEmptyOrWhitespace(errors, "username", "NotEmpty");
  if (member.getUsername().length() < 3 || member.getUsername().length() > 32) {
    errors.rejectValue("username", "Size.userForm.username");
  }

  if (memberService.findByUserName(member.getUsername()) != null) {
    errors.rejectValue("username", "Duplicate.userForm.username");
  }

  if (memberService.findByUserEmail(member.getEmail()) != null) {
    errors.rejectValue("email", "Duplicate.userForm.email");
  }

  ValidationUtils.rejectIfEmptyOrWhitespace(errors, "password", "NotEmpty");
  if (member.getPassword().length() < 7 || member.getPassword().length() > 32) {
    errors.rejectValue("password", "Size.userForm.password");
  }

  if (!member.getPasswordConfirm().equals(member.getPassword())) {
    errors.rejectValue("passwordConfirm", "Diff.userForm.passwordConfirm");
  }
```

오류 발생 시 rejectValue에서 두번째 파라미터 값의 내용을 출력한다. 
이는 validation.properties 파일에 정의되어 있다. 

```
NotEmpty=필수값입니다.
Size.userForm.username=이름은 3글자 이상 32자 미만으로 입력하세요
Duplicate.userForm.username=중복된 이름입니다
Size.userForm.password=패스워드는 7글자 이상 32자 미만으로 입력하세요
Diff.userForm.passwordConfirm=패스워드가 확인용 패스워드와 서로 맞지 않습니다
Duplicate.userForm.email=이미 사용 중인 이메일입니다
```

![validation error](validation-error.png)


### Reference 
<https://victorydntmd.tistory.com/328>