
# SpringBoot에 Swagger 적용하기
![swagger](swagger.png)

## Swagger

---
Swagger 공식 사이트 : <https://swagger.io/>

## 사용법 
spirngboot 프로젝트에 swagger 적용해보기 
pom.xml에 추가

```xml
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <!--버전을 넣지 않으면 동작하지 않는다. -->
    <version>2.9.2</version> 
</dependency>

<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <!--버전을 넣지 않으면 동작하지 않는다. -->
    <version>2.9.2</version>
</dependency>
```

SwaggerConfig 클래스 추가 
```java
package com.example.board;

import com.google.common.base.Predicate;
import com.google.common.base.Predicates;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@Configuration
@EnableSwagger2
public class SwaggerConfig {
    @Bean
    public Docket api(){
        return new Docket(DocumentationType.SWAGGER_2).select()
                .apis(Predicates.not(RequestHandlerSelectors.
                        basePackage("org.springframework.boot")))
                .paths(PathSelectors.any()).build();
    }
}
```

`localhost:8080/swagger-ui.html` 접속 