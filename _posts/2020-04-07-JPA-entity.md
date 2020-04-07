---
layout: post
title: "JPA - Entity에 대해"
excerpt: "JPA의 데이터베이스-객체 매핑 방법인 Entity 클래스에 대해 알아보자"
categories: [spring]]
comments: true
---

## 데이터베이스와 객체 매핑 방법 

### Entity 클래스 설정 
- Entity 클래스 : 데이터베이스 스키마의 내용을 자바 클래스로 표현할 수 있는 대상 
    - 해당 클래스에 `@Entity` 어노테이션을 선언해 엔티티 매니저가 관리해야 할 대상임을 표시 
        ```
        @Entity
        public class Project {
            ...
        }
        ```
- 주의 
    - 기본 생성자 필수 
        - 파라미터가 없는 public 또는 protected 생성자가 필요하다.
        - JPA spec으로 규정되어 있다.
        Why? JPA를 구현해서 쓰는 라이브러리들이 다양한 기술(Ex. Reflection)을 사용해서 객체를 프록싱할 때 필요하기 때문.
- Entity 속성
    - name
        - `@Entity(name = "Project")`
        - JPA에서 사용할 엔티티 이름을 지정
        - 기본값: 클래스 이름을 그대로 사용

@Entity 에노테이션을 사용할 때 실제 데이터베이스의 테이블명과 클래스명이 다른 경우에는 @Talbe 어노테이션을 추가해 매칠할 수 있다. 
```java
@Entity
@Table(name="tbl_proj")
Public class ProjectEntity{}
```

### 데이터베이스와 키 매핑 
데이터베이스의 기본키와 entity 간의 매핑을 위해 `@Id` 어노테이션을 사용한다. 

기본키 할당 방식에는 두 가지가 있다 
- **직접할당** : 기본키를 어플리케이션에서 직접 할당
    ```java
    @Id
    @Column(name = "id")
    private String id;
    ```

    ```java
    Board board = new Board();
    board.setId("id1");         //기본 키 직접 할당
    em.persist(board);
    ```
- **자동생성** : 데이터베이스가 자동으로 할당
    ex) 오라클 - sequence, MySQL - auto_increment
    데이터베이스 종류마다 sequence, auto_increment 등 기본키 자동생성을 지원하는 방법이 다르기 때문에, Spring Data JPA는 4가지 어노테이션을 지원한다.
    @Id 어노테이션 밑에 `@GeneratedValue(strategy = GenerationType.방식)`을 선언해 사용한다. 
    - **SEQUENCE** : 데이터베이스의 시퀀스 객체와 매핑
        - 주로 시퀀스를 지원하는 Oracle, PostgresSQL, DB2, H2에서 사용
        - @SequenceGenerator를 사용하여 시퀀스 생성기를 등록하고, 실제 데이터베이스의 생성될 시퀀스이름을 지정해줘야 함 
    - **IDENTITY** : 기본 키 생성을 데이터베이스에 위임 
        - 주로 MySQL, PostgresSQL, SQL Server, DB2에서 사용
    - **TABLE** : 키 값을 데이터베이스 테이블로 표현
        - 키 생성 전용 테이블을 하나 만들고 여기에 이름과 값으로 사용할 컬럼을 만드는 방법
        - 데이터베이스 종류와 상관없이 모든 데이터베이스에 적용 가능
        ```sql
        create table MY_SEQUENCES (
            sequence_name varchar(255) not null,
            next_val bigint,
            primary key (sequence_name)
        )
        ```
        ```java
        @Entity
        @TableGenerator(
            name = "BOARD_SEQ_GENERATOR",
            table = "MY_SEQUENCES",
            pkColumnValue = "BOARD_SEQ", allocationSize = 1)
        public class Board {

            @Id
            @GeneratedValue(strategy = GenerationType.TABLE,
                        generator = "BOARD_SEQ_GENERATOR")
            private Long id;
        }
        ```
        ```java
        private static void logic(EntityManger em) {
            Board board = new Board();
            em.persist(board);
            System.out.println("board.id = " + board.getId());
        }

        // 출력 : board.id = 1
        ```
        MY_SEQUENCES 테이블에 값이 없으면 JPA가 값을 INSERT 
    - **AUTO** : 데이터베이스에 따라서 IDENTITY, SEQUENCE, TABLE 중 하나를 자동으로 선택
        - 데이터베이스 종류에 의존적이지 않음 
        - 데이터베이스를 변경해도 코드를 수정할 필요 없음
        - ex) 오라클의 경우, SEQUENCE를 자동으로 선택해 사용
        ```java
        @Entity
        public class Project {

            @Id
            @GeneratedValue(strategy = GenerationType.AUTO)
            private Long projectId;
            private String projectName;
            private String projectSummary;
            ...
        ```




ref) 
<https://gmlwjd9405.github.io/2019/08/11/entity-mapping.html><br>
<https://ithub.tistory.com/24><br>
<https://ultrakain.gitbooks.io/jpa/chapter4/chapter4.6.html><br>

