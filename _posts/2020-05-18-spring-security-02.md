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
## 의존성 추가 
pom.xml에 라이브러리 추가 
```xml 
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

## Spring security 설정 
스프링부트에서 스프링 시큐리티 설정은 `WebSecurityConfigurerAdapter`라는 클래스를 상속받은 클래스에서 메서드를 오버라이딩 해 조정할 수 있다. 

SecurityConfig.java
```java

```


### Reference 
<https://victorydntmd.tistory.com/328>