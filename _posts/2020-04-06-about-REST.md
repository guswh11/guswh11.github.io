---
layout: post
title: "REST에 대해"
excerpt: "REST, RESTful, REST API에 대해 알아보자"
categories: [web]
comments: true
---

# About REST
![REST](rest.png)
>HTTP의 주요 저자 중 한 명인 로이 필딩이 웹 설계의 우수성에 비해 제대로 사용되어지지 못하는 모습에 안타까워하며<br>`웹의 장점을 최대한 활용할 수 있는 아키텍쳐`로써 발표함 

## 정의
- 자원을 이름으로 구분하여 해당 자원의 상태(정보)를 주고 받는 모든 것
    - 자원이란, 소프트웨어가 관리하는 모든 것<br>문서, 그림, 저장된 데이터(DBMS), 해당 소프트웨어 자체 등
    - DB의 학생 정보가 자원일 때, 'student'를 자원의 표현으로 정함

- HTTP 기반으로 자원에 접근하는 방식을 정의한 아키텍쳐 

- 월드 와이드 웹(www)과 같은 분산 하이퍼미디어 시스템을 위한 소프트웨어 개발 아키텍쳐

## 구성 
- **자원(Resource)**
    - URI 
- **행위(Verb)**
    - HTTP Method
- **표현(Respresentations)**

## 특성
- **클라이언트/서버 구조**
    - 클라이언트와 서버가 서로 독립적으로 구분되어야 함
    - 서버 또는 클라이언트 증설 시에 서로간의 의존성 때문에 확장에 문제가 되는 일이 없어야 함

- **상태 없음(Stateless)**
    - 통신 시에 서버는 클라이언트의 상태를 기억할 필요가 없음
    // HTTP는 비상태 프로토콜로, 
    // 클라이언트에서 상태 관리는 쿠키
    // 서버에서의 상태 관리는 세션 

- **레이어드 아키텍쳐(Layered Architecture)**
    - 서버와 클라이언트 사이에 게이트웨이, 방화벽, 프록시 등이 있는 것처럼 다계층 형태로 레이어를 추가, 수정, 제거할 수 있어야 함. 
    - 확장성 지님 

- **캐시(cache)**
    - REST는 HTTP가 가진 캐싱 기능 적용 가능
    - 캐시를 가질 경우, 클라이언트가 캐시를 통해서 응답을 재사용할 수 있음
    - 이를 통해 서버의 부하를 낮춤

- **코드 온 디멘드(Code on demand)**
    - 요청이 오면 코드를 줌
    - 특정 시점에 서버가 특정 기능을 수행하는 스크립트 또는 플러그인을 클라이언트에 전달해 해당 기능을 동작하도록 함
    ex) 자바스크립트

- **통합 인터페이스**
    - 서버와 클라이언트 간의 상호 작용은 일관된 인터페이스들 위에서 이뤄짐 

## REST 인터페이스 규칙 
- **리소스 식별**
    - 리소스는 URI와 같은 고유 식별자를 통해 표현됨

- **표현을 위한 리소스 처리**
    - 같은 데이터에 대해서 표현할 때, JSON, XML, HTML과 같이 다양한 콘텐츠 유형으로 표현 가능
    - 데이터 자체는 변경되지 않음

- **자기 묘사 메시지**
    - HTTP 통신 시, Http header에 메타 데이터 정보를 추가해 데이터에 대한 설명을 담을 수 있음

- **애플리케이션의 상태에 대한 하이퍼미디어**
    - 단순한 데이터 전달이 아닌 링크 정보를 포함해 웹을 구성

# RESTful
- REST 방식을 따르는 시스템을 RESTful이라 지칭
    - REST API를 제공하는 시스템을 'RESTful'하다고 할 수 있음 

- RESTful 하지 못한 경우
    - Ex1) CRUD 기능을 모두 POST로만 처리하는 API
    - Ex2) route에 resource, id 외의 정보가 들어가는 경우(/students/updateName)

# REST API
![REST API](restapi.png)

- REST 기반으로 서비스 API를 구현한 것

## REST API 설계
- **URI로 자원을 표현**
    - URI: Uniform Resource Identifier
    - 동사보사는 명사를 사용해 자원을 표현하는데 중점을 두어야 함(행위에 대한 표현이 들어가면 안됨)

- **자원에 대한 행위는 HTTP Method를 사용해 표현**
    - HTTP Method: GET / POST / PUT / DELETE등의 HTTP 프로토콜 사용 시의 호출 방식
    ```
    GET /members/delete/1 (X)
    DELETE /members/1 (O)
    ```
## URI 설계 규칙 
- '/'(슬래시)는 계층 관계를 나타내는 데 사용
    ```
    /리소스명/리소스 ID/관계가 있는 다른 리소스명
    ex) GET : /users/{userid}/devices 
    ```

- 마지막 문자로 '/'를 포함하지 않음

- '_'(언더바)보다는 '-'(하이픈)을 권장
    - 긴 경로 사용 시 '-'를 통해 가독성을 높임
    - '_'사용은 해석에 혼란을 줄 수 있음
    
- URI 경로에는 소문자가 적합 
- 파일 확장자는 포함시키지 않음 
    - Accept header를 사용<br>(컨텐츠 타입을 명시하는 Request type의 header)
    
    ``` 
    http://restapi.example.com/members/soccer/345/photo.jpg (X)
    GET / members/soccer/345/photo HTTP/1.1 Host: restapi.example.com Accept: image/jpg (O)
    ```

---

# Reference 

REST의 정의와 개념, REST API: <https://gmlwjd9405.github.io/2018/09/21/rest-and-restful.html>
<br>URI 설계 규칙: https://medium.com/@dydrlaks/rest-api-3e424716bab
<br>REST API 설계 규칙: <https://meetup.toast.com/posts/92>
<br>RESTful API 설계 가이드 (예제 존재): <https://sanghaklee.tistory.com/57>