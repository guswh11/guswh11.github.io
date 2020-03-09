---
layout: post
title: "Spring Boot Servlet 등록하기"
excerpt: "Servlet 사용을 위한 설정 등록 방법"
categories: [spring]
comments: true
image:
  feature: /capture-registering-servlet.png
---

Spring boot에서는 서블릿을 사용하기 위해 설정을 등록해야 한다. 

```java
@Configuration
public class ServletRegistrationConfig {

    @Bean
    public ServletRegistrationBean<NewServlet> getServletRegistrationBean(){
        ServletRegistrationBean<NewServlet> registrationBean = new ServletRegistrationBean<>(new NewServlet());

        registrationBean.addUrlMappings("/new");
        return registrationBean;
    }
}
```

NewServlet에는 등록하려는 서블릿이 들어가고,

`registrationBean.addUrlMappings("/new")` 에는 서블릿의 urlPattern을 넣어주면 된다. 

<br>
<br>

결과 코드는 위에 캡쳐해뒀다 ! 
<br>
<br>

---

<br>
이걸 몰라서 처음에 뭘 잘못 설정했나 하고 프로젝트를 몇번이나 다시 만들었다. 

책에 왜 안 써 두셨을까!?!?!?!?!?!?!??!?!?! 
