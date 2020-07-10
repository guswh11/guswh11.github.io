## 데이터베이스와 객체 매핑 방법 

### Entity 클래스 설정 
**`@Entity`**
- Entity 클래스 : 데이터베이스 스키마의 내용을 자바 클래스로 표현할 수 있는 대상 
    - 해당 클래스에 `@Entity` 어노테이션을 선언해 엔티티 매니저가 관리해야 할 대상임을 표시 
        ```java
        @Entity
        public class Project {
                    
            @Id
            @GeneratedValue(strategy = GenerationType.IDENTITY)
            private Long projectId;
            private String projectName;
            private String projectSummary;    

            @CreationTimestamp
            private Timestamp regDate;

            @UpdateTimestamp
            private Timestamp updateDate;

            @OneToOne(mappedBy = "project")
            private ProjectInfo projectInfo;

            @OneToMany(mappedBy = "project")
            private List<ProjectMember> projectMembers;
        }
        ```
        클래스 내부의 변수들은 각각 데이터베이스 테이블 내의 컬럼이 된다. 
- 주의 
    - 기본 생성자 필수 
        - 파라미터가 없는 public 또는 protected 생성자가 필요하다.
        - JPA spec으로 규정되어 있다.
        Why? JPA를 구현해서 쓰는 라이브러리들이 다양한 기술(Ex. Reflection)을 사용해서 객체를 프록싱할 때 필요하기 때문.
    - final 클래스, enum, interface, inner 클래스에는 사용할 수 없다. 
    - 저장할 필드에 final을 사용하면 안 된다. 
- 속성
    - name
        - `@Entity(name = "Project")`
        - JPA에서 사용할 엔티티 이름을 지정
        - 기본값: 클래스 이름을 그대로 사용
        - 패키지 간에 엔티티 이름이 겹치지 않도록 주의 

**`@Table`**
@Entity 에노테이션을 사용할 때 실제 데이터베이스의 테이블명과 클래스명이 다른 경우에는 @Table 어노테이션을 추가해 매칠할 수 있다. 
```java
@Entity
@Table(name="tbl_proj")
Public class ProjectEntity{}
```

### 필드와 컬럼 매핑
**@Column**
데이터베이스의 컬럼을 매핑하는 어노테이션
-  속성
    - name
        데이터베이스 칼럼과 엔티티 내 변수를 각기 다른 이름으로 매핑할 때 사용
        ```java
        @Column(name="project_id")
        private Long projectId;
        ```
    - insertable
        엔티티 저장 시 해당 필드를 저장할 것인지 결정한다. false로 설정하면 데이터베이스에 컬럼으로 저장하지 않는다. 
    - updatable
        컬럼 수정(update) 시 데이터베이스에 이를 반영할 것인지 여부를 결정한다. 
        ```
        @Column(updatable = false)
        ```
        위와 같이 설정하면 컬럼을 수정해도 데이터베이스에 반영되지 않는다. 
    - nullable
        ```
        @Column(nullable = false)
        ```
        위와 같은 설정은 데이터베이스에서의 NOT NULL과 같다. 
    - unique
    - columnDefinition
        직접 컬럼 정보를 작성할 수 있다. 
        ```
        @Column(columnDefinition = "varchar(100) default 'EMPTY'")
        ```
    - length
        문자 길이 제약 조건으로, String 타입에만 사용 가능하다. 
    - precision, scale(DDL)
        BigDecimal 타입에서 사용한다.(BigInteger 가능) precision은 소수점을 포함한 전체 자리수이고, scale은 소수점 자릿수이다. double랑 float타입에는 적용 되지 않는다.

**@Enumerated**
Enum 타입을 매핑할 때 사용한다. 
아래는 일반 사용자와 관리자 구분을 하는 UserRole enum에 대한 예제이다. 
```java
/**
 * Created by yun_dev1 on 2016-12-26.
 */
public enum UserRole {
    USER, //0
    ADMIN //1
}
```
위 enum을 사용하는 entity에서의 선언은 다음과 같이 한다. 
```java
@Column(name="role")
@Enumerated(EnumType.ORDINAL)
private UserRole role;
```

**@Temporal**
날짜 타입을 매핑할 때 사용한다. 
- 타입
    - DATE: 날짜, DB의 date 타입과 매핑
    ex) 2019-08-13
        ```
        @Temporal(TemporalType.DATE)
        ```

    - TIME: 시간, DB의 time 타입과 매핑
    ex) 14:20:38
    - TIMESTAMP: 날짜와 시간, DB의 timestamp 타입과 매핑
    ex) 2019-08-13 14:20:38

>>java 8의 LocalDate(date), LocalDateTime(timestamp) 을 사용할 때는 생략 가능

**@Lob**
DB에서 varchar를 넘어서는 큰 내용을 넣고 싶은 경우에 사용
- 필드의 길이 제한이 없다. 
- @Lob에는 지정할 수 있는 속성이 없다.
- 매핑하는 필드의 타입에 따라 DB의 BLOB, CLOB과 매핑된다.

**@Transient**
특정 필드를 컬럼에 매핑하지 않음.
- DB에 관계없이 메모리에서만 사용하고자 하는 객체에 해당 annotation을 사용
- 해당 annotation이 붙은 필드는 DB에 저장, 조회가 되지 않는다.

**리스너**
JPA의 리스너 기능은 entity의 생명주기에 따른 이벤트를 처리할 수 있도록 한다. 각 어노테이션은 다음과 같은 시점에 해당 메서드를 호출한다. 

- **@PrePersist**

    @PrePersist는 persist() 메서드를 호출해 entity를 영속성 컨텍스트에 관리하기 직전에 해당 메서드를 호출한다.
    날짜와 같이 매핑 시에 값 참조를 위해서 미리 인스턴스가 생성되어야 하는 필드에 사용한다. 
    
        ```java
        private Date date;

        @PrePersist
        public void setDate(){
            date = new Date();
        }
        ```

- **@PreUpdate**

    flush나 commit을 호출해서 엔티티를 데이터베이스에 수정하기 직전 호출 된다. 

- **@PostLoad**

    entity가 영속성 컨텍스트에 조회된 직후 또는 refresh()를 호출한 후 호출 된다. 

- **@PreRemove** 

    remove() 메서드를 호출해서 엔티티를 영속성 컨텍스트에서 삭제하기 직전에 호출.<br>또한 삭제 명령어로 영속성 전이가 일어날 때도 메서드를 호출되게 한다.<br>orphanRemoval에 대해서는 flush나 commit시 호출 된다.

- **@Postpersist**

    flush나 commit을 호출해서 엔티티를 데이터베이스에 저장한 직후에 호출된다.<br>식별자가 항상 존재한다.<br>참고로 식별자 생성 전략이 IDENTITY면 식별자를 생성하기 위해 persist()를 호출한 직후에 바로 Postpersist가 호출 된다.

- **@PostUpdate**

    flush나 commit을 호출해서 엔티티를 데이터베이스에 수정한 직후에 호출 된다.

- **@PostRemove**

    flush나 commit을 호출해서 엔티티를 데이터베이스에 삭제한 직후에 호출된다.

>>Entity 클래스 내부에서 각 리스너 어노테이션은 여러 번 선언될 수 없다.

**이름 매핑 전략 변경하기**
단어를 구분할 때 자바는 주로 카멜 표기법을 사용하고, 데이터베이스는 언더스코어(_)를 사용한다.<br>데이터베이스 칼럼과 엔티티 내 변수를 각기 다른 이름으로 매핑하려면 `@Column.name` 속성을 사용해야 한다. 
name 속성을 사용해 직접 매핑 전략을 구현해도 되지만, `hibernate.ejb.naming_strategy` 속성을 사용하면 이름 매핑 전략을 변경할 수 있다. 이 클래스는 테ㅣ블 명이나 컬럼 명이 생략되면 자바의 카멜 표기법을 테이블의 언더스코어 표기법으로 매핑한다. 
```java
<property name="hibernate.ejb.naming_strategh"
    value="org.hibernate.cfg.ImprovedNamingStrategh">
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

**Reference**
<https://gmlwjd9405.github.io/2019/08/11/entity-mapping.html><br>
<https://ithub.tistory.com/24><br>
<https://ultrakain.gitbooks.io/jpa/chapter4/chapter4.6.html><br>
<http://wonwoo.ml/index.php/post/995>
