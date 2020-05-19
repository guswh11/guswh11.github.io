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
스프링부트에서 스프링 시큐리티 설정은 `WebSecurityConfigurerAdapter`라는 클래스를 상속받은 클래스에서 메서드를 오버라이딩 해 조정할 수 있다. 

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