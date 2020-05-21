---
layout: post
title: "Spring security를 이용한 회원가입/로그인/로그아웃 예제"
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

## 모델 구현 
유저 정보를 담을 Member entity와 Role enum 객체는 다음과 같다. 
*Member.java*

```java
@Getter
@Setter
@Entity
@Table(name = "tbl_member")
@ToString
public class Member extends AbstractEntityModel{

  @Id
  @GeneratedValue(strategy=GenerationType.IDENTITY)
  private long id;
  private String username;

  private String password;
  private String passwordConfirm;
  @Unique
  private String email;

  @Column(name = "role_name")
  @Enumerated(EnumType.STRING)
  private MemberRole role;
}
```

*MemberRole.java*

```java
public enum MemberRole {
    USER, //0
    ADMIN //1
}
```

## 컨트롤러 구현 
*MemberController.java*

```java
@Slf4j
@Controller
public class MemberController {
	@Autowired
	private MemberService memberService;
	@Autowired
	private MemberValidator memberValidator;

	@RequestMapping(value = "/signup", method = RequestMethod.GET)
	public String registration(Model model) {
		model.addAttribute("userForm", new Member());

		return "signup";
	}

	@RequestMapping(value = "/signup", method = RequestMethod.POST)
	public String registration(@ModelAttribute("userForm") @Valid Member userForm, BindingResult bindingResult) {

		memberValidator.validate(userForm, bindingResult);

		if (bindingResult.hasErrors()) {
			log.debug("valid error");
			return "signup";
		}

		userForm.setRole(MemberRole.USER);
		memberService.save(userForm);
		log.debug("userInfo" + userForm.toString());
		log.debug("email" + userForm.getEmail() + "|" + userForm.getPassword());

		return "redirect:/welcome";
	}

	@RequestMapping(value = "/login", method = RequestMethod.GET)
	public ModelAndView login(Model model, String error, String logout) {
		if (error != null)
			model.addAttribute("error", "아이디 또는 패스워드가 잘못 되었습니다.");

		if (logout != null)
			model.addAttribute("message", "로그아웃 되었습니다.");

		return new ModelAndView("login");
	}


	@RequestMapping(value = {"/"}, method = RequestMethod.GET)
	public String home(Model model) {
		return "home";
	}

	@GetMapping("/403")
	public String error403() {
		return "/error/403";
	}

	@GetMapping("/welcome")
	public String welcome(){
		return "welcome";
	}
}
```

## 서비스 구현 
*MemberServiceImpl.java*

```java
@Service("MemberService")
@Slf4j
public class MemberServiceImpl implements MemberService{
	@Autowired
	private MemberRepository memberRepository;
	@Autowired
	private BCryptPasswordEncoder bCryptPasswordEncoder;

	@Override
	public Member findByUserEmail(String email) {
		return memberRepository.findByEmail(email);
	}

	@Override
	public Member findByUserName(String username) {
		return memberRepository.findByUsername(username);
	}

	@Override
	public void save(Member member) {
		member.setPassword(bCryptPasswordEncoder.encode(member.getPassword()));
		member.setPasswordConfirm(bCryptPasswordEncoder.encode(member.getPasswordConfirm()));
		memberRepository.save(member);
	}
}
```

- **findByUserEmail(), findByUserEmail()**
	- 유저의 상세 정보를 조회하는 메서드로, Member entity를 반환한다. 
- **save()**
	- 회원가입을 처리하는 메서드
	- 비밀번호를 암호화 해 저장한다.
		- 데이터베이스에 저장하기 직전에 암호화를 진행한다. 
		- BCryptPasswordEncoder는 스프링 시큐리티에서 제공하는 단방향 암호화 클래스이다. 


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

## HTML 구현 
*header.html*

```html
...
<li class="user-menu" sec:authorize="isAnonymous()">
	<a href="/login">로그인</a>
</li>
<li class="user-menu" sec:authorize="isAnonymous()">
	<a href="/signup">회원가입</a>
</li>
<li class="user-menu" sec:authorize="isAuthenticated()">
	<a href="/logout">로그아웃</a>
</li>
...
```

home 페이지의 header 부분 코드의 일부이다.<br>
`sec:authorize="isAnonymous()"`을 통해 미인증 유저에게는 로그인과 회원가입 메뉴를 보여주고,<br>`sec:authorize="isAuthenticated()"`을 통해 인증된 유저에게만 로그아웃 메뉴를 보여준다. 

|![home-authenticated](home-authenticated.png)|![home-not-authenticated](home-not-authenticated.png)|
|:--:|:--:|
| *authenticated* | *not authenticated* |

*signup.html*

```html
<body>
<div class="container">
	<form class="form-horizontal" role="form" method="POST" action="/signup" th:object="${userForm}">
		<div class="row">
			<div class="col-md-3"></div>
			<div class="col-md-6">
				<h2>회원가입</h2>
				<hr/>
			</div>
		</div>
		<div class="row">
			<div class="col-md-3 field-label-responsive">
				<label for="username">이름</label>
			</div>
			<div class="col-md-6">
				<div class="form-group">
					<div class="input-group mb-2 mr-sm-2 mb-sm-0">
						<div class="input-group-addon" style="width: 2.6rem"><i class="fa fa-user"></i></div>
						<input type="text" name="username" class="form-control" id="username"
							   placeholder="이름"/>
					</div>
					<div class="col-lg-4">
						<span class="label label-danger" th:if="#{#fields.hasErrors('username')}" th:errors="*{username}">User Name Error</span>
					</div>
				</div>
			</div>
		</div>

		<!--email-->
		<div class="row">
			<div class="col-md-3 field-label-responsive">
				<label for="email">이메일</label>
			</div>
			<div class="col-md-6">
				<div class="form-group">
					<div class="input-group mb-2 mr-sm-2 mb-sm-0">
						<div class="input-group-addon" style="width: 2.6rem"><i class="fa fa-user"></i></div>
						<input type="text" name="email" class="form-control" id="email"
							   placeholder="my@info.com"/>
					</div>
					<div class="col-lg-4">
						<span class="label label-danger" th:if="#{#fields.hasErrors('email')}" th:errors="*{email}">email Error</span>
					</div>
				</div>
			</div>
		</div>

		<!--password-->
		<div class="row">
			<div class="col-md-3 field-label-responsive">
				<label for="password">패스워드</label>
			</div>
			<div class="col-md-6">
				<div class="form-group">
					<div class="input-group mb-2 mr-sm-2 mb-sm-0">
						<div class="input-group-addon" style="width: 2.6rem"><i class="fa fa-user"></i></div>
						<input type="password" name="password" class="form-control" id="password"/>
					</div>
					<div class="col-lg-4">
						<span class="label label-danger" th:if="#{#fields.hasErrors('password')}" th:errors="*{password}">password Error</span>
					</div>
				</div>
			</div>
		</div>

		<!--password confirm-->
		<div class="row">
			<div class="col-md-3 field-label-responsive">
				<label for="passwordConfirm">패스워드 확인</label>
			</div>
			<div class="col-md-6">
				<div class="form-group">
					<div class="input-group mb-2 mr-sm-2 mb-sm-0">
						<div class="input-group-addon" style="width: 2.6rem"><i class="fa fa-user"></i></div>
						<input type="password" name="passwordConfirm" class="form-control" id="passwordConfirm"/>
						<input type="hidden" th:name="${_csrf.parameterName}" th:value="${_csrf.token}"/>
					</div>
					<div class="col-lg-4">
						<span class="label label-danger" th:if="#{#fields.hasErrors('passwordConfirm')}" th:errors="*{passwordConfirm}">passwordConfirm Error</span>
					</div>
				</div>
			</div>
		</div>


		<div class="row">
			<div class="col-md-3"></div>
			<div class="col-md-6">
				<button type="submit" class="btn btn-success"><i class="fa fa-user-plus"></i> 회원등록</button>
			</div>
		</div>
	</form>
</div>
</body>
```

스프링 시큐리티에서는 CSRF 공격 방지를 위해 token validation을 제공하는데, 이 토큰 값을 넘기기 위해 다음과 같은 작업도 진행한다. 

```html
<input type="hidden" th:name="${_csrf.parameterName}" th:value="${_csrf.token}"/>
```

화면에서는 hidden으로 보이지 않고, 서버에서 값을 비교한다. 

스프링 시큐리티가 적용되면 POST 방식으로 보내는 모든 데이터는 csrf 토큰 값이 필요하다.<br>
토큰 값이 없는 상태에서 form 전송을 할 경우, 컨트롤러에 다음과 같이 POST 메서드를 매핑할 수 없다는 에러가 발생한다.<br>
`error : HttpRequestMethodNotSupportedException: Request method 'POST'`

*login.html*

```html
<body>
<div class="container">
	<div class="row">
		<div class="col-md4 col-md-offset-4">
			<div class="login-panel panel panel-default">
				<div class="panel-heading">
					<h3 class="panel-title">로그인</h3>
				</div>

				<div th:if="${param.error}">
					<div class="alert alert-danger">
						<th:block th:text="${error}"></th:block>
					</div>
				</div>

				<div th:if="${param.logout}">
					<h1 style="color:blue">Logged out.</h1>
				</div>

				<div class="panel-body">
					<form role="form" method="post" th:action="@{sign-in}">
						<fieldset>
							<div class="form-group">
									<input class="form-control" placeholder="E-mail" name="email" type="email"
										   autofocus="autofocus"/>

							</div>
							<div class="form-group">
								<input class="form-control" placeholder="Password" name="passwd" type="password"
									   value=""/>
							</div>
							<div class="checkbox">
								<label>
									<input name="remember-me" type="checkbox"/>Remember me
								</label>
							</div>
							<input type="hidden" th:name="${_csrf.parameterName}" th:value="${_csrf.token}"/>
							<button type="submit" class="btn btn-lg btn-success btn-block">login</button>
						</fieldset>
					</form>
				</div>
			</div>
		</div>
	</div>
</div>

<th:block layout:fragment="script">

	<script th:inline="javascript">
        console.log($.fn.jquery);
	</script>

</th:block>
</body>
```

회원가입 페이지와 마찬가지로, 로그인 정보 제출 시 csrf 토큰 값을 넘기는 작업을 진행한다.

`welcome.html`

```html
<div layout:fragment="content">

	<div class="container">
		<h2> home </h2>
		<span sec:authentication="name">userName</span>
		role:<span sec:authentication="principal.authorities"></span>
		<a th:href="@{/logout}">sign out</a>
	</div>

</div>
<!--  fragmentsment -->

<th:block layout:fragment="script">

	<script th:inline="javascript">

	</script>

</th:block>
```

회원가입과 로그인 성공 후 보이는 welcome 페이지다.<br>
`sec:authentication="name"`을 통해 username 값을,<br>
`sec:authentication="principal.authorities"`을 통해 유저의 role 값을 가져온다. 

![welcome](welcome.png)

`sec:authentication`과 `sec:authorize`의 사용법은 아래 링크에서 확인할 수 있다.<br>
<https://www.thymeleaf.org/doc/articles/springsecurity.html>

### Reference 
<https://victorydntmd.tistory.com/328>