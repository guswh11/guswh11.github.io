---
layout: post
title: "JPA - Repository에 대해"
excerpt: "JPA의 Repository에 대해 알아보자"
categories: [spring]
comments: true
permalink: /articles/:year-:month/:title
tags:
  - JPA
  - Spring Boot
  - Database
---

## Repository 
Repository는 Entity 조작에 필요한 쿼리를 메서드화해서 사용할 수 있는 역할을 한다. 

![JpaRepository class diagram](JpaRepository.png)

Repository 인터페이스는 Spring Data JPA에서 제공하는 미리 만들어진 Repository 인터페이스들을 상속받는다.
상속을 통해 다음과 같은 기능을 제공한다.

| method | 기능 | 
|--------|------|
| save() | 레코드 저장 (insert, update) |
| findOne() | 기본키로 레코드 한 건 찾기 |
| findAll() | 전체 레코드 불러오기 | 
| count() | 레코드 갯수 반환 | 
| delete() | 레코드 삭제 | 

_findOne()은 fineById()로 교체된 것 같다_

### 사용 방법 
아래는 JpaRepository를 상속하고 ProjectEntity를 조작하는 Repository이다. 
```java
@Repository
public interface ProjectRepository extends JpaRepository<Project, Long> {

}
```
첫 번째 인자로는 해당 Entity 클래스명을, 두 번째 인자로는 ID 타입이 될 수 있는 Long(Int)을 입력한다. 

만들어진 Repository는 다음과 같은 방식으로 이용할 수 있다. 
```java
@RestController
public class ProjectInfoController {
    @Autowired
    ProjectInfoRepository projectInfoRepository;

    @PostMapping("/project-info/{id}")
    public ProjectInfo getProjectInfo(@PathVariable String id){
        Long projectId = Long.parseLong(id);

        Optional<ProjectInfo> getProjectInfo = projectInfoRepository.findById(projectId);
        ProjectInfo projectInfo = getProjectInfo.get();
        return projectInfo;
    }
}
```
상속을 통해 제공되는 기본 기능을 제외한 다른 기능을 추가하고 싶으면, 규칙에 맞는 사용자 정의 메서드를 추가해야 한다.
규칙은 다음과 같다.

| method | 설명 |
|--------|------|
| findBy로 시작 | 쿼리를 요청하는 메서드 |
| countBy로 시작 | 쿼리 결과 레코드 수를 요청하는 메서드 | 

- **findBy** method 예제
아래와 같은 방식으로 findBy로 시작되는 메서드를 정의하고 사용할 수 있다. 
    ```java
    @Repository
    public interface UserRepository extends JpaRepository<UserEntity, Long>{
        UserEntity findByName(String name);
        Page<UserEntity> findByName(String name, Pageable pageable); 
    }
    ```
    ```java 
    @SpringBootApplication
    public class JPAMain{
            public static void main(String[] args) {
                    ...
                    UserEntity user = userRepository.findByName("조현진");
                    ...
            }
    }
    ```
    메서드의 반환형이 객체면 하나의 결과만을 전달하고,<br>반환형이 List라면 쿼리 조건에 맞는 모든 객체를 전달한다. 
    
- **countBy** method 예제
아래 메서드는 사용 시 targetId에 해당하는 row 수를 반환한다. 
    ```java
    @Repository
    public interface UserRepository extends JpaRepository<UserEntity, Long>{
        Integer countByTargetId(@Param("targetId") Long targetId);
    }
    ```

메소드에 포함할 수 있는 키워드는 다음과 같다.

| 키워드 | 예시 | 설명 |
| ----- | ------| ----|
| And | findByEmailAndUser(String email, String userId) | 여러 필드를 and로 검색 |
| Or | findByEmailOrUser(String email, String userId) | 여러 필드를 or로 검색 |
| Between |  findByCreatedAtBetween(Date fromDate, Date toDate) | 필드의 두 값 사이에 있는 항목 검색 |
| LessThan |  findByAgeGraterThanEqual(int age) | 작은 항목 검색 | 
| GreaterThanEqual | findByAgeGreaterThanEqual(int age) | 크거나 같은 항목 검색 |
| Like | findByNameLike(String name) | like 검색(부분적으로 일치하는 칼럼 검색) |
| IsNull | findByJobIsNull() | Null인 항목 검색 |
| In | findByJob(String ...jobs) | 여러 값 중에 속하는 항목 검색 |
| OrderBy |  findByEmailOrderByNameAsc(String email) | 검색 결과를 정렬해 전달 | 


- **@Query**
Repository에서 @Query를 선언해 직접 sql을 입력할 수도 있다. 
    ```java
    public interface UserRepository extends JpaRepository<User, Long> {

    @Query("select u from User u where u.emailAddress = ?1")
    User findByEmailAddress(String emailAddress);
    }
    ```
- **Pageable**
    Query 메소드의 입력변수로 아래와 같이 Pageable 변수를 추가하면 Page타입을 반환형으로 사용할 수 있다.

    Pageable 객체를 통해 페이징과 정렬을 위한 파라미터를 전달한다.

    >Pageable 인스턴스와 page, size, sort 파라미터를 사용해 페이징을 구현할 수 있다. 

    ```java
    @Repository
    public interface UserRepository extends JpaRepository<UserEntity, Long>{
        UserEntity findByName(String name);
        Page<UserEntity> findByName(String name, Pageable pageable); 
    }
    ```
    Pageable 입력 변수는 Controller에서부터 전달받아야 한다.
    
    ```java
    @RestController
    @RequestMapping("/member")
    public class UserController {
        @Autowired
        UserService userService; 

        @RequestMapping("")
        Page<User> getUsers(Pageable pageable){
            return userService.getList(pageable)
        }
    }
    ```

>Repository의 쿼리 메소드나 어노테이션에 대한 전체 가이드는 [여기](https://docs.spring.io/spring-data/jpa/docs/1.10.1.RELEASE/reference/html/#jpa.sample-app.finders.strategies)에서 확인할 수 있다. 





