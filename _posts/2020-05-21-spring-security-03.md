---
layout: post
title: "Spring security - Remember-Me로 로그인 유지하기"
categories: [spring]
comments: true
tags:
  - Spring Boot
  - Spring Security
---

## Remember-Me
Remember-me는 인증 정보를 쿠키 형태로 보관해서 만료 기간 전까지 저장된 쿠키 정보로 인증을 대신 처리해 주는 기능이다.

이를 위해 Remember-me는 쿠키를 하나 더 사용해, 인증 시 쿠키에 암호화한 인증정보를 담아서 넣어놓는다. 이 값은 세션이 만료되었거나, 세션이 없을 때 사용한다. 즉, 해당 요청에 해당하는 세션을 찾지 못할 때, 같이 보내온 Remember-me 쿠키가 있으면 그 쿠키에 들어있는 인증정보로 인증을 시도한다. 인증이 성공하면 새로운 세션 id와 쿠키가 발급된다. 

이러한 경우 문제점이 하나 있다. 로그인 시 쿠키를 탈취 당할 수 있다. 로그아웃시 세션과 쿠키를 모두 날려줘야한다.

좀 더 안전한 방법이 있다. 랜덤한 토큰값(매번 바뀜)을 사용하고, 랜덤한 시리즈(고정)라는 것을 사용한다. 토큰은 매번 인증할 때마다 새로 바뀌고, 시리즈는 인증을 해도 바뀌지 않는다.

이 경우 해커가 쿠키를 탈취하면, 사용자가 인증하려고 할 때, 유효하지 않은 토큰과 유효한 시리즈의 username으로 접속한다. 이 경우 토큰이 유효하지 않기 때문에 모든 토큰을 삭제하여 해커가 더 이상 탈취한 쿠키를 사용하지 못하도록 방지해준다. 그 후, form 기반으로 다시 로그인한다.

### RememberMe 토큰
쿠키가 생성되고 데이터베이스에도 저장될 수 있도록 entity를 생성해준다. 

*RememberMeToken.java*

```java
@Entity
@Table(name = "persistent_logins")
@Getter
@Setter
@NoArgsConstructor
public class RememberMeToken implements Serializable{
    @Id
    @Column(name = "SERIES")
    private String series;

    @Column(name = "USERNAME", nullable = false)
    private String username;

    @Column(name = "TOKEN", nullable = false)
    private String token;

    @Column(name = "LAST_USED", nullable = false)
    private Date lastUsed;

    public RememberMeToken(PersistentRememberMeToken token) {
        this.series = token.getSeries();
        this.username = token.getUsername();
        this.token = token.getTokenValue();
        this.lastUsed = token.getDate();
    }
}
```

### RememberMeTokenService
위에서 생성한 entity를 데이터베이스에 저장하기 위한, PersistentTokenRepository를 상속받은 서비스이다. 

*RememberMeTokenService.java*

```java
@Service
@Transactional
@Slf4j
public class RememberMeTokenService implements PersistentTokenRepository {

    @Autowired
    private RememberMeTokenRepository rememberMeTokenRepository;

    @Override
    public void createNewToken(PersistentRememberMeToken token) {
        RememberMeToken newToken = new RememberMeToken(token);
        this.rememberMeTokenRepository.save(newToken);
    }

    @Override
    public void updateToken(String series, String tokenValue, Date lastUsed) {
        RememberMeToken token = this.rememberMeTokenRepository.findBySeries(series);
        if (token != null){
            token.setToken(tokenValue);
            token.setLastUsed(lastUsed);
            this.rememberMeTokenRepository.save(token);
        }
    }

    @Override
    public PersistentRememberMeToken getTokenForSeries(String seriesId) {
        RememberMeToken token = this.rememberMeTokenRepository.findBySeries(seriesId);
        if(token != null){
            return new PersistentRememberMeToken(token.getToken(), token.getSeries(), token.getToken(), token.getLastUsed());
        }
        return null;
    }

    @Override
    public void removeUserTokens(String username) {
        List<RememberMeToken> tokens = this.rememberMeTokenRepository.findByUsername(username);
        if(tokens != null){
            this.rememberMeTokenRepository.delete(tokens);
        }
    }
}
```

### Spring Security 설정
*SecurityCongih.java*
RememberMeTokenService를 스프링 시큐리티와 연결하기 위한 설정이다. 

```java
public class SecurityConfig extends WebSecurityConfigurerAdapter {
	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http
        ...
				.rememberMe()
				.key("jpub")
				.rememberMeParameter("remember-me")
				.rememberMeCookieName("jpubcookie")
				.tokenValiditySeconds(86400) //1day
				.tokenRepository(rememberMeTokenService()).userDetailsService(myUserService())
        ...
	}

  @Bean
	public RememberMeTokenService rememberMeTokenService() throws Exception{
		return new RememberMeTokenService();
	}
}
```

- **key()**
  - Remember-me 인증을 위해 발행되는 토큰을 구별하는 키 이름을 지정
- **tokenValiditySeconds()**
  - 기본 토큰 유효시간 설정
  - 별도의 지정이 없을 시 기본 2주동안 유효하다.
- **rememberMeParameter()**
  - 로그인 페이지의 html 소스에서 `input` 태그의 name에 해당한다.
  
  ```html
  <input name="remember-me" type="checkbox"/>Remember me
  ```

### Reference
<https://velog.io/@max9106/Spring-Security-RememberMe%EB%A1%9C%EA%B7%B8%EC%9D%B8-%EC%9C%A0%EC%A7%80%ED%95%98%EA%B8%B0>