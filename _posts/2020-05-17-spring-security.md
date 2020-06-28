---
layout: post
title: "Spring security에 대해"
categories: [spring]
comments: true
tags:
  - Spring
  - Spring Security
---

# Spring security
## Spring security(스프링 시큐리티)란 
스프링 시큐리티는 스프링 기반의 어플리케이션의 보안(인증과 권한)을 담당하는 프레임워크로,
인증과 권한 제어에 필요한 필터의 모음이다.<br>
필터를 기반으로 권한에 따른 접근 제어 기능, 패스워드 암호화 등의 기능을 지원한다.<br>
만약 스프링 시큐리티를 사용하지 않는다면 개발자가 보안 관련 로직을 직접 작성해야 할 것이다.
> 웹 클라이언트의 요청에 대해서 필요한 사전 작업이 있을 경우에 필터를 사용(서블릿보다 먼저 동작함)


#### 보안 관련 용어
- **접근 주체(Principal)** : 보호된 대상에 접근하는 유저
    - 스프링 시큐리티에서 인증 후 데이터베이스에서 가져온 사용자 정보 객체는 principal 메서드로 읽어올 수 있다.
- **인증(Authenticate)** : 입력받은 정보와 사용자 정보를 비교해서 신원을 확인하는 것(ex. 로그인)
    - 애플리케이션의 작업을 수행할 수 있는 주체임을 증명
    - 코드에서의 Authentication: 인증 과정에서 사용되는 핵심 객체
    - 스프링 시큐리티는 기본적으로 HTTP 기본 인증 방식으로 동작 
- **인가(Authorization)** : 사용자가 가진 권한을 확인하여 권한에 맞게 접근을 허용하는 것
    - 인증 과정을 통해 사용자를 식별한 후 이루어지는 절차이다. 
    - 스프링 시큐리티에서는 권한 관리가 Role(접속한 사용자가 사이트에서 수행하는 역할) 기반으로 이루어진다. 

## 스프링 시큐리티의 구조
### 인증 처리 구조
![authentication-architecture](spring-security-architecture.png)

스프링 시큐리티는 세션-쿠키방식으로 인증한다.
아래는 인증 처리 과정이다. 
1. 유저가 로그인을 시도 (http request)
    - username과 password를 조합한 UsernamePasswordAuthenticationToken 인스턴스가 생성됨
2. 토큰이 검증을 위해 AuthenticationManager의 인스턴스로 전달됨 
3. 인증이 성공하면(DB에 있는 유저라면) AuthenticationManager는 Authentication 인스턴스를 리턴하고, UserDetailsService를 통해 유저의 세션 생성
4. 생성된 유저 세션은 인메모리 세션저장소인 SecurityContextHolder에 저장
5. 유저에게 세션 ID와 함께 응답을 내려줌
6. 이후 요청에서는 요청쿠키에서 JSESSIONID를 검증 후, 유효하면 Authentication를 쥐어줌

### 스프링 시큐리티의 filter, SecurityFilterChain
스프링 시큐리티와 관련된 서블릿 필터도 실제로는 연결된 여러 필터들로 구성되어있다.<br>
이러한 필터들의 흐름과 모임을 Chain이라고 표현한다. 

필터 체인의 동작 방식은 다음과 같다. 

![filter chain](filter-chain.png)

아래는 스프링 시큐리티의 필터 체인이다. 

![spring security filter chain](spring-security-filter-chain.png)

이미지 출처: <http://atin.tistory.com/590>

- **SecurityContextPersistenceFilter** : SecurityContextRepository에서 SecurityContext를 가져오거나 저장하는 역할을 한다.
- **LogoutFilter** : 설정된 로그아웃 URL로 오는 요청을 감시하며, 해당 유저를 로그아웃 처리
- **(UsernamePassword)AuthenticationFilter** : (아이디와 비밀번호를 사용하는 form 기반 인증) 설정된 로그인 URL로 오는 요청을 감시하며, 유저 인증 과정(로그인)을 진행한다. 
    - AuthenticationManager를 통한 인증 실행
    - 인증 성공 시, 얻은 Authentication 객체를 SecurityContext에 저장 후 AuthenticationSuccessHandler 실행
    - 인증 실패 시, AuthenticationFailureHandler 실행
- **DefaultLoginPageGeneratingFilter** : 인증을 위한 로그인폼 URL을 감시한다.
    - 사용자가 별도의 로그인 페이지를 구현하지 않은 경우, 스프링에서 기본적으로 설정한 로그인 페이지를 처리한다. 
- **BasicAuthenticationFilter** : HTTP 기본 인증 헤더를 감지해 처리하고, 결과를 SecurityContextHolder에 저장한다. 
- **RememberMeAuthenticationFilter** : SecurityContext에 인증 객체가 있는지 확인하고 RememberMeServices를 구현한 객체의 요청이 있을 경우, Remember-Me를 인증 토큰으로 컨텍스트에 주입합니다.
- **RequestCacheAwareFilter** : 로그인 성공 후, 원래 요청 정보를 재구성하기 위해 사용된다.
- **SecurityContextHolderAwareRequestFilter** : HttpServletRequestWrapper를 상속한 SecurityContextHolderAwareRequestWapper 클래스로 HttpServletRequest 정보를 감싼다.    
    - SecurityContextHolderAwareRequestWrapper 클래스는 필터 체인상의 다음 필터들에게 부가정보를 제공한다.
- **AnonymousAuthenticationFilter** : 이 필터가 호출되는 시점까지 사용자 정보가 인증되지 않았다면 인증토큰에 사용자가 익명 사용자로 나타난다.
- **SessionManagementFilter** : 인증된 사용자와 관련된 모든 세션을 추적
    - 인증된 사용자일 경우 SessionAuthenticationStrategy를 호출하여 세션 고정 보호 메커니즘을 활성화하거나, 여러 동시 로그인을 확인하는 것과 같은 세션 관련 활동을 수행한다.
- **ExceptionTranslationFilter** : 필터 체인 내에서 발생하는 모든 예외를 위임하거나 전달하는 역할을 한다.
- **FilterSecurityInterceptor** : HTTP 리소스의 보안 처리를 수행 

### SecurityContextHolder, SecurityContext, Authentication
세 클래스는 스프링 시큐리티의 주요 컴포넌트이다. 
각 컴포넌트의 관계를 간단히 표현하자면 다음과 같다.

![SecurityContextHolder, SecurityContext, Authentication](spring-security-components.png)

유저가 로그인 정보를 입력한 후 인증에 성공하면, 사용자의 principal과 credential(자격 증명을 위한 암호화된 정보) 정보는 Authentication에 담긴다.<br>
스프링 시큐리티는 Authenticaton을 SecurityContext에 보관하고, 
SecurityContext는 내부 메모리인 SecurityContextHolder에 담겨 보관된다. 

#### Authentication 
모든 접근 주체(유저)는 Authentication을 생성한다.

```java
public interface Authentication extends Principal, Serializable {
    Collection<? extends GrantedAuthority> getAuthorities(); // Authentication 저장소에 의해 인증된 사용자의 권한 목록
    Object getCredentials(); // 주로 비밀번호
    Object getDetails(); // 사용자 상세정보
    Object getPrincipal(); // 주로 ID
    boolean isAuthenticated(); //인증 여부
    void setAuthenticated(boolean isAuthenticated) throws IllegalArgumentException;
}
```

#### AuthenticationManager
![AuthenticationManager](authentication-manager.png)

유저의 요청 내에 담긴 Authentication를 AuthenticationManager에 넘겨주면, AuthenticationManager를 구현한 ProviderManager가 인증 절차를 처리한다.<br>ProviderManager는 `private List<AuthenticationProvider> providers;`로 여러 AuthenticationProvider를 가질 수 있는데, 이 provider가 인증 처리를 거친 후 Authentication 를 반환해준다.
- **AuthenticationManager** : 인증요청을 받고 Authentication를 채운다.
- **AuthenticationProvider** : 실제 인증이 일어나며, 성공하면 Authentication.isAuthenticated = true 를 한다.

### Reference
<https://docs.spring.io/spring-security/site/docs/4.2.7.RELEASE/reference/htmlsingle/#getting-started><br>
<https://sjh836.tistory.com/165><br>
<https://coding-start.tistory.com/153>
