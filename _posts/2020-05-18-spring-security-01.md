---
layout: post
title: "Spring security 설정 방법"
categories: [spring]
comments: true
permalink: /articles/:year-:month/:title
tags:
  - Spring
  - Spring Security
---

# Spring security 시작하기 
## Spring security 설정 방법 
### 의존성 추가
`pom.xml`에 다음과 같이 추가
```xml
<!-- Properties  -->
<security.version>4.2.7.RELEASE</security.version>

<!-- Security -->
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-core</artifactId>
    <version>${security.version}</version>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-web</artifactId>
    <version>${security.version}</version>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-config</artifactId>
    <version>${security.version}</version>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-taglibs</artifactId>
    <version>${security.version}</version>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-test</artifactId>
    <version>${security.version}</version>
</dependency>
```

LDAP, OpenID등의 추가적인 기능을 사용한다면 다음 링크에서 찾아 알맞게 추가하면 된다. 
[Section 2.4.3, “Project Modules”](https://docs.spring.io/spring-security/site/docs/4.2.7.RELEASE/reference/htmlsingle/#modules)

### web.xml 설정 
```xml
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>
        classpath:applicationContext.xml
        classpath:applicationContext-security.xml
    </param-value>
</context-param>

<!-- Spring Security -->
<listener>
    <listener-class>org.springframework.security.web.session.HttpSessionEventPublisher</listener-class>
</listener>
<filter>
    <filter-name>springSecurityFilterChain</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
</filter>
<filter-mapping>
    <filter-name>springSecurityFilterChain</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

- **HttpSessionEventPublisher** : 한 유저가 다른 브라우저로 동시 로그인하는 것을 막음
- **DelegatingFilterProxy** : 모든 요청은 이 프록시필터를 거친다. 스프링시큐리티는 이를 통해 인증, 인가를 수행

### security 설정
설정을 보면 알겠지만, dispatcher-context 와는 별개의 설정(context)이다. 만약 dispatcher-context 에서 component-scan 을 하고있었다면, security의 bean들은 이들을 의존성주입받을 수 없다. (서비스 이하 컴포넌트들은 applicationContext 에서 component-scan 하는 걸 추천)

**applicationContext-security.xml**
```xml
<security:http auto-config="true" use-expressions="true">
    <security:csrf disabled="true"/> 
    <security:intercept-url pattern="/**" access="permitAll" /> 
    <security:form-login login-page="/login" authentication-success-handler-ref="loginSuccessHandler" authentication-failure-handler-ref="loginFailureHandler" login-processing-url="/auth" username-parameter="id" password-parameter="pw" /> 
    <security:logout logout-url="/logout" invalidate-session="true" logout-success-url="/login?status=logout" /> 
    <security:session-management invalid-session-url="/login"> 
        <security:concurrency-control max-sessions="1" error-if-maximum-exceeded="false" /> 
    </security:session-management> 
</security:http> 

<!-- secured method --> 
<security:global-method-security secured-annotations="enabled" /> 

<!-- provider --> <security:authentication-manager> 
    <security:authentication-provider ref="userAuthHelper" /> 
</security:authentication-manager> 

<bean id="loginSuccessHandler" class="com.devljh.domain.user.helper.LoginSuccessHandler"> 
    <property name="defaultTargetUrl" value="/main" /> 
    <property name="alwaysUseDefaultTargetUrl" value="true" /> 
</bean> 
<bean id="loginFailureHandler" class="com.devljh.domain.user.helper.LoginFailureHandler"> 
    <property name="defaultFailureUrl" value="/login?status=fail" /> 
</bean> 

<bean id="userAuthService" class="com.devljh.domain.user.UserAuthService" /> 
<bean id="passwordEncoder" class="org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder" /> 

<bean id="userAuthProvider" class="com.devljh.domain.user.helper.UserAuthProvider"> 
    <property name="userDetailsService" ref="userAuthService" /> 
    <property name="passwordEncoder" ref="passwordEncoder" /> 
</bean>
```

## security.xml의 설정
### security:http (springSecurityFilterChain 설정)
- **auto-config** : true 로 할경우 filter는 dafault 값으로 동작한다. 
false 면 anonymous, x509, http-basic, session-management, expression-handler, custom-filter, port-mappings, request-cache, remember-me 를 정의해줘야한다.
- **use-expressions** : 스프링 표현식(spEL)의 사용여부다
- **csrf** : csrf 보안 설정여부
- **intercept-url** : pattern에 정의된 경로에 대해 access 권한을 지정 (Filter가 감시)
- **form-login**
    - **login-page** : login form 페이지 URL
    - **username-parameter** : form id의 name 속성값
    - **password-parameter** : form pw의 name 속성값
    - **login-processing-url** : form action 값 (security를 이용해 인증처리)
    - **default-target-url** : 로그인 성공 시 이동 URL
    - **authentication-failure-url** : 로그인 실패 시 이동 URL
    - **always-use-default-target** : true 로 하면 무조건 default-target-url 로 이동한다. 
    false 로 하면 authentication-success-handler 대로 redirect 된다.
    - **authentication-success-handler-ref** : 로그인 성공 시 프로세스 정의, bean id 입력
        - 만약 최종 로그인일시를 DB에 기록해야한다면 handler로 처리하는게 좋겠다.
        - res.sendRedirect 등으로 처리
    - **authentication-failure-handler-ref** : 로그인 실패 시 프로세스 정의, bean id 입력
- **logout**
    - **logout-url** : 로그아웃 처리할 URL 
    (security가 알아서 만들기 때문에, 이 경로로 요청만 하면된다)
    - **logout-success-url** : 로그아웃 성공 시 이동 URL
    - **success-handler-ref** : 로그아웃 성공 시 프로세스 정의, bean id 입력
    - **invalidate-session** : 로그아웃 시 세션 삭제여부
- **session-management**
    - **invalid-session-url** : invalid session(세션 타임아웃 등) 일 때 이동 URL
    - **max-sessions** : 최대 허용 가능한 세션 수
    - **expired-url** : 중복 로그인 발생시 이동 URL (처음 접속한 세션이 invalidate가 되고 다음 요청시 invalid-session-url로 이동)
    - **error-if-maximum-exceeded** : max-sessions을 넘었을때 접속한 세션을 실패처리할지 여부 (expired-url와 얘중에 하나만 쓰면 될듯)

### AuthenticationSuccessHandler, AuthenticationFailureHandler, LogoutSuccessHandler
이들은 위 설정에서 보았듯이, 각각 로그인성공 시, 로그인실패 시, 로그아웃 성공 시 동작하게 할 수 있는 handler 이다. 이들은 인터페이스로 내부 코드들은 org.springframework.security.web.authentication 에 있다. 이걸 직접 구현해도 되지만, 이 패키지가보면 Simple* 이라는 아주좋은 샘플 구현체들이 이미 있다. 쌩으로 구현하는 것보단 당연히 상속받는 걸 추천한다. 이것들을 상속받으면 xml에서 설정자 주입으로 default-url 등을 설정 할 수 있다. (인터페이스를 구현했으면 직접 sendRedirect로 처리했어야함)

### PasswordEncoder
암호화 알고리즘을 골라서 bean을 만들 수 있다. 
AuthenticationProvider의 구현체로 DaoAuthenticationProvider를 사용할 경우, 기본적으로 PlaintextPasswordEncoder(=NoOpPasswordEncoder)를 사용한다.

```java
public DaoAuthenticationProvider() {
    this.setPasswordEncoder((PasswordEncoder)(new PlaintextPasswordEncoder()));
}
```

- **NoOpPasswordEncoder** : deprecate 되었다. 테스트할 때만 쓰자
- **BCryptPasswordEncoder** : bcrypt 해쉬 알고리즘을 이용 (추천)
- **StandardPasswordEncoder** : sha 해쉬 알고리즘을 이용

### access
- **hasRole(Role)** : 해당 Role 을 갖고있는 사용자 허용
- **hasAnyRole(Role1, Role2, ...)** : 해당 Role 중에 1개이상 갖고있는 사용자 허용
- **anonymous** : 익명 사용자 허용
- **authenticated** : 인증된 사용자 허용
- **permitAll** : 모든 사용자 허용
- **denyAll** : 모든 사용자 허용

## 유용한 annotation
- **@Secured** : 각 요청경로에 따른 권한 설정은 위의 xml에서도 할 수 있지만, 메소드레벨에서 @Secured 를 통해서도 할 수 있다. `<security:global-method-security secured-annotations="enabled"/>` 를 추가하면 된다.
    - @Secured("ROLE_ADMIN") , @Secured({"ROLE_ADMIN","ROLE_USER"})
    - 비인가자 접근 시 AccessDeniedException 던짐
- **@PreAuthorize** : 위와 비슷하지만 spEL 을 사용할 수 있다. `<security:global-method-security pre-post-annotations="enabled" />` 를 설정한다.
    - @PreAuthorize("hasRole('ADMIN')")
- **@PostAuthorize** : 위와 동일
- **@AuthenticationPrincipal** : 컨트롤러단에서 세션의 정보들에 접근하고 싶을 때 파라미터에 선언해준다.
    - public ModelAndView userInfo(@AuthenticationPrincipal User user)
    - 이거 안쓰고 확인하려면 (User) SecurityContextHolder.getContext().getAuthentication().getPrincipal(); 이런식으로 써야한다.
    - 4.x 부터는 org.springframework.security.core.annotation.AuthenticationPrincipal 을 import 해야한다.
    - mvc:annotation-driven.mvc:argument-resolvers 에 bean 으로 `org.springframework.security.web.method.annotation.AuthenticationPrincipalArgumentResolver`를 등록한다.

### Reference
<https://docs.spring.io/spring-security/site/docs/4.2.7.RELEASE/reference/htmlsingle/#getting-started>
<https://sjh836.tistory.com/165>
