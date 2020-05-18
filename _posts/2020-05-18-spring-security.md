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
보안과 관련해서 체계적으로 많은 옵션들을 지원한다. 
만약 스프링 시큐리티를 사용하지 않는다면 개발자가 보안 관련 로직을 직접 작성해야 할 것이다.  
스프링 시큐리티는 필터(filter) 기반으로 동작하기 때문에 spring MVC와 분리되어 관리 및 동작한다. 
> filter란


#### 보안 관련 용어
- **접근 주체(Principal)** : 보호된 대상에 접근하는 유저
- **인증(Authenticate)** : 현재 유저가 누구인지 확인(ex. 로그인)
    - 애플리케이션의 작업을 수행할 수 있는 주체임을 증명
    - 코드에서의 Authentication: 인증 과정에서 사용되는 핵심 객체
- **인가(Authorize)** : 해당 리소스에 대해 접근 가능한 권한을 가지고 있는지 확인하는 과정(인증 이후)
- **권한(Authorization)** : 인가 과정에서 해당 리소스에 대한 제한된 최소한의 권환을 가졌는지 확인
    - 권한 승인이 필요한 부분으로 접근하려면 인증 과정을 통해 주체가 증명 되어야만 한다
    - 권한 부여에도 두가지 영역이 존재하는데 웹 요청 권한, 메소드 호출 및 도메인 인스턴스에 대한 접근 권한 부여

## 스프링 시큐리티의 구조 
### 인증 관련 구조
![authentication-architecture](spring-security-architecture.png)

**spring security는 세션-쿠키방식으로 인증한다.**
1. 유저가 로그인을 시도 (http request)
2. AuthenticationFilter 에서부터 위와같이 user DB까지 타고 들어감
3. DB에 있는 유저라면 UserDetails 로 꺼내서 유저의 session 생성
4. spring security의 인메모리 세션저장소인 SecurityContextHolder 에 저장
5. 유저에게 session ID와 함께 응답을 내려줌
6. 이후 요청에서는 요청쿠키에서 JSESSIONID를 검증 후, 유효하면 Authentication를 쥐어준다.

### 스프링 시큐리티의 filter
![filter](spring-security-filter.png)

- **SecurityContextPersistenceFilter** : SecurityContextRepository에서 SecurityContext를 가져오거나 저장하는 역할을 한다. (SecurityContext는 밑에)
- **LogoutFilter** : 설정된 로그아웃 URL로 오는 요청을 감시하며, 해당 유저를 로그아웃 처리
- **(UsernamePassword)AuthenticationFilter** : (아이디와 비밀번호를 사용하는 form 기반 인증) 설정된 로그인 URL로 오는 요청을 감시하며, 유저 인증 처리
    - AuthenticationManager를 통한 인증 실행
    - 인증 성공 시, 얻은 Authentication 객체를 SecurityContext에 저장 후 AuthenticationSuccessHandler 실행
    - 인증 실패 시, AuthenticationFailureHandler 실행
- **DefaultLoginPageGeneratingFilter** : 인증을 위한 로그인폼 URL을 감시한다.
- **BasicAuthenticationFilter** : HTTP 기본 인증 헤더를 감시하여 처리한다.
- **RequestCacheAwareFilter** : 로그인 성공 후, 원래 요청 정보를 재구성하기 위해 사용된다.
- **SecurityContextHolderAwareRequestFilter** : HttpServletRequestWrapper를 상속한 SecurityContextHolderAwareRequestWapper 클래스로 HttpServletRequest 정보를 감싼다.    
    - SecurityContextHolderAwareRequestWrapper 클래스는 필터 체인상의 다음 필터들에게 부가정보를 제공한다.
- **AnonymousAuthenticationFilter** : 이 필터가 호출되는 시점까지 사용자 정보가 인증되지 않았다면 인증토큰에 사용자가 익명 사용자로 나타난다.
- **SessionManagementFilter** : 이 필터는 인증된 사용자와 관련된 모든 세션을 추적한다.
- **ExceptionTranslationFilter** : 이 필터는 보호된 요청을 처리하는 중에 발생할 수 있는 예외를 위임하거나 전달하는 역할을 한다.
- **FilterSecurityInterceptor** : 이 필터는 AccessDecisionManager 로 권한부여 처리를 위임함으로써 접근 제어 결정을 쉽게해준다.

### Authentication 
모든 접근 주체(유저)는 Authentication을 생성한다. 
이것은 SecurityContext 에 보관되고 사용된다. 
즉 security의 세션들은 내부 메모리(SecurityContextHolder)에 쌓고 꺼내쓰는 것이다.

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

### AuthenticationManager
![AuthenticationManager](authentication-manager.png)

유저의 요청 내에 담긴 Authentication를 AuthenticationManager 에 넘겨주고, AuthenticationManager 를 구현한 ProviderManager가 처리한다. 정확히는 ProviderManager 는 private `List<AuthenticationProvider> providers;` 로 여러 AuthenticationProvider를 가질 수 있는데, 이 친구들이 처리를 해서 Authentication 를 반환해준다. (실패하면 예외던짐)
- **AuthenticationManager** : 인증요청을 받고 Authentication를 채운다.
- **AuthenticationProvider** : 실제 인증이 일어나며, 성공하면 Authentication.isAuthenticated = true 를 한다.

### Reference
<https://docs.spring.io/spring-security/site/docs/4.2.7.RELEASE/reference/htmlsingle/#getting-started>
<https://sjh836.tistory.com/165>
<https://coding-start.tistory.com/153>
