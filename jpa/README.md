# **JPA 이해하기** 

## **[JPA 시작하기](https://github.com/keepinmindsh/tech-course/blob/main/jpa/start/README.md)**

## **H2Database 설정** 

https://www.h2database.com/html/main.html 를 이용한 H2 Database 구성 

h2 database는 다른 RDB의 환경 구성 및 설치에 있어 훨씬 쉽고 간편하기 때문에 개발 시점에서 
활용하기 좋은 db이다. 구성 방식에 따라서 memory에 올려구성할 수도 있으며, 아래의 이미지와 같이 Web에서 쉽게 접근하여 데이터 조회 및 질의에 수월하다.  

![H2 Database 접속 화면](https://github.com/keepinmindsh/tech-course/blob/5836f80ab528b6ab5d8f2cc2f0c4be6333c8a1f8/assets/jpastudy_0001.png)

## **JPA(Hibernate)의 Get Started**

Spring Data JPA가 아닌 hibernate를 활용하여 JPA에 대해서 알아보고자 한다.   
Spring Data JPA를 사용할 경우 EntityManager의 선언 필요 없디 Repository를 활용 가능하기 때문에 Hibernate의 entityManager를 직접 선언하여 사용하면서 JPA를 이해해본다. 

- JPA Java Project 구성시 Maven 모듈 ( Gradle Or Maven )
    - https://mvnrepository.com/artifact/org.hibernate/hibernate-entitymanager
    - https://mvnrepository.com/artifact/com.h2database/h2

```gradle

plugins {
    id 'java'
}

group 'org.example'
version '1.0-SNAPSHOT'

repositories {
    mavenCentral()
}

dependencies {

    // https://mvnrepository.com/artifact/com.h2database/h2
    implementation 'com.h2database:h2:2.0.204'
    // https://mvnrepository.com/artifact/org.hibernate/hibernate-entitymanager
    implementation 'org.hibernate:hibernate-entitymanager:6.0.0.Alpha7'

    testImplementation 'org.junit.jupiter:junit-jupiter-api:5.7.0'
    testRuntimeOnly 'org.junit.jupiter:junit-jupiter-engine:5.7.0'
}

test {
    useJUnitPlatform()
}

```

```java

package bong.lines.sample;

import javax.persistence.*;

public class HelloWorldJPA {
    public static void main(String[] args) {

        EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("hello");

        EntityManager entityManager = entityManagerFactory.createEntityManager();

        EntityTransaction entityTransaction = entityManager.getTransaction();

        entityTransaction.begin();

        try{

            Member member = new Member();
            member.setId(2L);
            member.setName("Hello");
            entityManager.persist(member);

        }catch (Exception exception){
            entityTransaction.rollback();
        }finally {
            entityManager.close();
        }

        entityManagerFactory.close();
    }
}

```

## **Persistence.xml**

- persistence.xml 설정 
    - jpa의 경우, 기본적인 설정 파일을 필요로 함. 순수 자바 프로젝트에서 사용할 경우 META-INF 하위에 persistence.xml 를 위치시키는 것이 중요함.
    - 항목별 설명
        - hibernate.dialect : 데이터베이스 방언, 다양하게 존재하는 DB의 DDL,DML 등의 특성, 고유한 기능에 맞춰 사용할 수 있게 해주는 설정
    - javax. ~ : 표준에서 제정한 것 
    - hibernate. ~ : 하이버네이트 전용 옵션
    - hibernate.show_sql : query를 출력함
    - hibernate.format_sql : sql을 예쁘게
    - hibernate.use_sql_comments : 쿼리 실행사유를 표시

```xml

<?xml version="1.0" encoding="UTF-8"?>
<persistence version="2.2"
             xmlns="http://xmlns.jcp.org/xml/ns/persistence" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd">
    <persistence-unit name="hello">
        <class>bong.lines.sample.Member</class>
        <class>bong.lines.sample.MemberForSeq</class>
        <properties>
            <!-- 필수 속성 -->
            <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
            <property name="javax.persistence.jdbc.user" value="sa"/>
            <property name="javax.persistence.jdbc.password" value=""/>
            <property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/test"/>
            <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>

            <!-- 옵션 -->
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.format_sql" value="true"/>
            <property name="hibernate.use_sql_comments" value="true"/>
            <property name="hibernate.hbm2ddl.auto" value="update" />
        </properties>
    </persistence-unit>
</persistence>

```


## **Persistence Context**

- 객체와 관계형 데이터베이스 매핑하기
- **영속성 컨텍스트**
    - JPA를 이해하는데 가장 중요한 용어임. 
    - "엔티티를 영구 저장하는 환경"이라는 의미로 논리적인 개념이다. 
    - Entity Manager를 통해 영속성 컨텍스트에 접근함. 
    - **1차 캐시**
        - Entity 조회, 1차 캐시
        - 객체를 조회할 때, 영속성 컨텍스트에서 1차 캐시를 질의하여 데이터가 존재한다면 캐시에 있는 값을 그대로 조회함
        - 하나의 Transaction 안에서만 동작하기 때문에 효율성의 이점이 크지 않음. 다만 비즈니스 로직이 복잡할 경우는 도움이 될 수 있음
        - 객체에 대해서 저장하는 persist 시점에도 1차 캐시로 등록이 가능하다.
    - **동일성 보장**
        - 1차 캐시로 반복 가능한 읽기(REPEATABLE READ) 등급의 트랜잭션 격리 수준을 데이터 베이스가 아닌 애플리케이션 차원에서 제공
    - **트랜잭션을 지원하는 쓰기 지연**
        - persist 시점에 데이터를 DB Insert 처리하지 않고 Commit 시점에 처리된다.
        - persist를 다수 호출 후에 commit 명령어에 의해서 실행되므로, jdbc batch insert 와 동일하게 동작한다.
        - bufferning을 모아서 적용 가능
    - **변경 감지 ( Dirty Checking )**
        - 영속 컨텍스트에 의해서 Transaction Commit (flush) 시점에 entity와 snapshot를 비교하는데, entity와 snapshot을 비교하여 변경된 값에 대해서 update query를 생성하여 commit 시점에    database에 반영된다.
    - **지연 로딩**

## **Hibernate Method 활용** 

#### QUERY - JPQL 방언에 맞춰서 Paging 등의 쿼리 적용이 가능함.

```java

public static void main(String[] args) {
    List<Member> memberList = entityManager.createQuery("select m from Member as m", Member.class)
            .setFirstResult(1)
            .setMaxResults(10)
            .getResultList();
    memberList.forEach(
            item -> {
                System.out.println("name : " + item.getName());
            }
    );
}

```

#### JPA를 통해서 객체를 반환하면 해당 객체는 JPA에서 관리하여 변경사항을 Transaction Commit시 반영한다.

```java

public static void main(String[] args) {
    Member findMember = entityManager.find(Member.class, 1L);
    findMember.setName("Good!");
}

```

#### 삭제 처리 

```java

public static void main(String[] args) {
    Member findMember = entityManager.find(Member.class, 1L);
    entityManager.remove(findMember);
}

```

#### 조회 클래스 분리

```java

public static void main(String[] args) {
    Member findMember = entityManager.find(Member.class, 1L);
    System.out.println("Id :" + findMember.getId() + ", Name : " + findMember.getName());
}

```

#### 저장 클래스 분리

```java

public static void main(String[] args) {
    Member member = new Member();
    member.setId(2L);
    member.setName("Hello");
    entityManager.persist(member);
}            

```

#### 비영속, 영속 상태

```java

public static void main(String[] args) {
    // 비영속
    Member member = new Member();
    member.setId(100L);
    member.setName("Bong JPA");

    // 영속 상태
    // persist 시점에는 실제로 데이터가 저장되는 시점이 아님.
    entityManager.persist(member);
}   

```

#### 1차 캐시 

 - 영속성 컨텍스트로 구성되면 1차 캐시의 역할로 이미 메모리에 로딩되어 있는 객체에 대해서는 DB로 별도의 질의 없이 조회가 가능하다. 

```java
public static void main(String[] args) {
    // TODO 1차 캐시
    Member member = new Member();
    member.setId(101L);
    member.setName("Hello");
    entityManager.persist(member);

    Member member1 = entityManager.find(Member.class, 101L);
    Member member2 = entityManager.find(Member.class, 101L);
}   
```

#### 동일성 보장 

```java

public static void main(String[] args) {
    Member member1 = entityManager.find(Member.class, 101L);
    Member member2 = entityManager.find(Member.class, 101L);

    System.out.println("member2 = " + ( member1 == member2));
}   

```

#### Transaction 쓰기 지연

```java

public static void main(String[] args) {
    Member member1 = new Member(150L, "A");
    Member member2 = new Member(160L, "B");

    entityManager.persist(member1);
    entityManager.persist(member2);
}   

```

#### 변경 감지 

```java

public static void main(String[] args) {
    Member member = entityManager.find(Member.class, 150L);
    member.setName("ZZZZZ")
}   

```

## **Database 스키마 자동 설정**

- DLL을 애플리케이션 실행 시점에 자동 생성 
- 테이블 중심 -> 객체 중심 
- 데이터 베이스 방언을 활용해서 데이터베이스에 맞는 적절한 DDL, DML 생성 
    - 데이터베이스 방언별로 달라지는 것 확인 필요 
- 이렇게 생성된 DDL은 개발장비에서만 사용 
- hibernate.hbm2ddl.auto
    - create : 기존 테이블 삭제후 다시 생성 ( drop 하고 create가 됨 )
    - create-drop : create와 같으나 종료 시점에 테이블 DROP
    - update : 변경 분만 반영 ( 운영 db에서 절대 사용하면 안됨 )
    - validate : 엔티티와 테이블이 정상 매핑되었는지만 확인 - 확인 용도 
    - none : 사용하지 않음 
- DDL 생성 기능 
    - 제약조건 추가 
        - @Column(nullable = false, name= "", unique = true, length = 10 )
            - DDL 생성을 도와주는 기능을 제공함. 실행 로직에는 영향을 주지 않음 

```shell

hibernate.hbm2ddl.auto=create 일 때, @Column(nullable = false, length = 500, unique = false) 는 아래와 같이 동작함.

Hibernate: 
    
    drop table if exists MBR cascade 
Hibernate: 
    
    create table MBR (
       id bigint not null,
        name varchar(500) not null,
        primary key (id)
    )

```

## Mapping Annotations

- @Column - 컬럼 매핑
    - name
    - insertable : 등록 여부
    - updatable : 변경 여부
    - nullable : null 값의 허용 여부 설정
    - unique : @Table의 uniqueConstraint와 같이만 한 컬럼에 간단히 유니크 제약조건을 걸 때 사용한다.
잘 사용하지 않는 값 - unique 인덱스 생성시 , 유니크 제약조건을 이름이 반영이 어려워 @Table 에 같이 선언하여 사용한다.
- @Table(uniqueConstraint= )
    - columnDefinition
        - 컬럼 정의를 직접하고 싶은 경우,
    - length
        - 문자길이 제약 조건, String 타입에 사용한다.
    - precision
    - BigDecimal 타입에서 사용한다. 아주 큰 숫자나 소수점 사용할 경우
- @Temporal - 날짜 타입 매핑
    - DATE : 날짜, 데이터 베이스 date 타입과 매핑 ( 2021-01-12 )
    - TIME : 시간, 데이터 베이스 time 타입과 매핑 ( 11:11:11 )
    - TIMESTAMP : 날짜와 시간, 데이터베이스 timestamp 타입과 매핑 ( 2021-01-12 11:11:11 )
    - 최근에는 ~ LocalDate(년월)와 LocalDateTime ( 년월일) - 최신 버전의 경우 변수 선언서 데이터 타입을 LocalDate와 LocalDateTime를 적용하는 경우 그에 맞게 호출됨. 아래의 test1과 test2를 참고할 것
- @Enumerated - Enum Type 활용
    - 기본값 : ORDINAL - Enum의 순서를 저장하는 것
        - 쓰면 문제가 되는 이유는, enum에 값을 정의할 때, 내부에서 정해지는 enum의 index 값에 따라 순번이 변경되어 이슈가 발생할 수 있음
        - 운영상에서는 ORDINAL를 사용하지 않음
        - 반드시 String으로 EnumType을 사용할 것
    - @Lob - Clob, Blob
        - @Lob에는 지정할 수 있는 속성이 없다.
        - 매핑하는 필드 타입이 문자면 CLOB 매핑, 나머지는 BLOB 매핑
            - CLOB : String, char[], java.sql.CLOB
            - BLOB : byte[], java.sql.BLOB 
- @Transient - 매핑을 안하고 싶을 경우

```java
import javax.persistence.*;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.util.Date;

@Entity
//@Table(name = "MBR")
public class Member {
    @Id
    private Long id;

    @Column(nullable = false, length = 500, unique = false, name = "name")
    private String name;

    private Integer age;
    
    @Enumerated(EnumType.STRING)
    private RoleType roleType;
    
    @Temporal(TemporalType.TIMESTAMP)
    private Date createDAte;

    @Temporal(TemporalType.TIMESTAMP)
    private Date lastModifiedDate;

    private LocalDate test1;
    private LocalDateTime test2;
    
    @Lob
    private String description;

    public Member() {
    }
}
```

```shell
create table Member (
   id bigint not null,
    age integer,
    createDAte timestamp(6),
    description clob,
    lastModifiedDate timestamp(6),
    name varchar(500) not null,
    roleType varchar(255),
    test1 date,
    test2 timestamp(6),
    primary key (id)
)
```

## 기본기 매핑 

- 생성 전략 
    - IDENTITY
    - SEQUENCE
    - TABLE 
    - AUTO 
- IDENTITY 
    - 기본키 생성을 데이터 베이스에 위임
    - 주로 MySQL, PostgreSQL, SQL Server, DB2 에서 사용
    - JPA에서 IDENTITY 전략을 제외하고는 모두 트랜잭션 커밋 시점에 INSERT SQL을 실행, 그 이유는 기본키를 미리 생성하기 때문임. 
    - IDENTITY 전략의 경우 em.persist() 시점에 INSERT_SQL이 실행되어 DB에서 기본키를 반환함. SELECT가 추가로 발생하지 않음 

```java

@Entity
@SequenceGenerator(name="member_seq_generator", sequenceName = "member_seq", allocationSize = 1)
public class MemberForSeq {
    
  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY, generator = "member_seq_generator")
  private Long id;

}

```

- SEQUENCE 
    - 데이터 베이스 시퀀스는 유일한 값을 순서대로 생성하는 특별한 데이터 베이스 오브젝트 ( 예 : 오라클 시퀀스 )
    - 오라클, PostgreSQL, DB2, H2 데이터베이스에서 사용
    - JPA에서 IDENTITY 전략을 제외하고는 모두 트랜잭션 커밋 시점에 INSERT SQL을 실행, 그 이유는 기본키를 미리 생성하기 때문임. 
    - SEQUENCE에서 타입을 정의할 때, Long을 사용해야는 이유는 10억이 넘어가는 경우 Long으로 변경하는 것이 힘들기 때문에 Long을 사용하는 것이 나음

```java

@Entity
@SequenceGenerator(name="member_seq_generator", sequenceName = "member_seq", allocationSize = 1)
public class MemberForSeq {

  @Id
  @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "member_seq_generator")
  private Long id;

}

```

- TABLE 
    - 키 생성 전용 테이블을 하나 만들어서 데이터베이스 시퀀스를 흉내내는 전략 
        - name : 식별자 생성기 이름
        - table : 키 생성 테이블 명
        - pkColumnName : 시퀀스 컬럼명
        - valueColumnName : 시퀀스 값 컬럼명
        - pkColumnValue : 키로 사용할 값 이름
        - initialValue : 초기값, 마지막으로 생성된 값이 기준이다.
        - allocationSize : 시퀀스 한번 호출에 증가하는 수(성능 최적화에 사용됨)
        - catalog, schema : 데이터베이스 catalog, schema 이름
        - uniqueConstraint : 유니크 제약 조건을 지정할 수 있다.
    - 장점 : 모든 데이터베이스에 적용 가능
    - 단점 : 성능

```java

@Entity
@TableGenerator(
        name = "MEMBER_SEQ_GENERATOR",
        table = "MY_SEQUENCE",
        pkColumnValue = "MEMBER_SEQ",
        allocationSize = 1
)
public class MemberForSeq {

  @Id
  @GeneratedValue(strategy = GenerationType.TABLE, generator = "member_seq_generator")
  private Long id;

}

```

## **연관관계 매핑**

- 양방향 매핑 
  - 양쪽으로 참조해서 값을 가질 수 있을 경우 
    - 객체의 참조와 테이블의 외래키의 차이
      - 객체의 경우, 로딩하는 객체 내에서 연관관계를 가지는 객체까지 로딩하므로, 각 객체에 연관관계를 가지는 객체의 정보를 포함한다. 
      - 테이블의 경우, 주요키와 외래키의 관계에 의해서 하나의 키에 의해서 연관관계가 연결된다. 

- 연관관계의 주인과 mappedBy
  - 객체와 테이블 간에 연관관계를 맺는 차이를 이해한다.
    - 객체 연관관계 2개 
      - 회원 -> 팀 연관관계 1개 ( 단방향 )
      - 팀 -> 회원 연관관계 1개 ( 단방향 )
    - 테이블 연관관계 = 1 개 
      - 회원 <-> 팀의 연관관계 1개 ( 양방향 )

- 객체의 양방향 관계 
  - 객체의 양방향 관계는 사실 양방향 관계가 아니라 서로 다른 단방향 관계 2개다. 
  - 객체를 양방향으로 참조하려면 단방향 연관관계 2개를 만들어야 한다. 

- 테이블의 양방향 관계 
  - 테이블은 외래키 하나로 두 테이블의 연관관계를 관리 
  - MEMBER.TEAM_ID 외래키 하나로 양방향 연관관계 가짐 ( 양쪽으로 조인할 수 있다. )


- 딜레마를 해결하는 방법
  - 딜레마 : Team 객체와 Member 객체에 존재하는 teamId를 어느 곳에서든 업데이트 할 수 있는데.. 어디서 해야하지?
  - Member와 Team의 객체 내에 존재하는 member의 값을 어떻게 관리해야 하는가?
    - 둘 중 하나로 외래키를 관리해야 한다.

- 연관관계의 주인 
  - 양방향 매핑 규칙 
    - 객체의 두 관계 중 하나를 연관관계의 주인으로 지정
    - **연관관계의 주인만이 외래키를 관리 ( 등록, 수정 )**
    - **주인이 아닌 쪽은 읽기만 가능**
    - 주인은 mappedBy 속성 사용 불가 
    - 주인이 아닌면 mappedBy 속성으로 주인 지정
      - mappedBy의 속성의 값에 대해서는 값을 수정하는 것은 변경 불가

- 그렇다면 누구를 주인으로? 
  - **외래키가 있는 곳을 주인으로 정해라** 
    - ManyToOne의 Many가 연관관계의 주인이 되고 
    - N:1의 N이 주인 
  - 여기서는 Member.team이 연관관계의 주인 
    - 진짜 매핑 - 연관관계의 주인 Member.team 
    - 가짜 매핑 - 주인의 반대편 Team.members


- Entity의 기본 정의는 아래와 같이 !

```java
 
@Entity
public class Member {

    @Id
    @GeneratedValue
    @Column(name="MEMBER_ID")
    private Long id;

    @Column(name="USERNAME")
    private String username;

    /**
     * 여러 N의 Team에 하나의 Member
     */
    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;
}


@Entity
public class Team {

  @Id
  @GeneratedValue
  @Column(name="TEAM_ID")
  private Long teamId;

  private String name;

  @OneToMany(mappedBy = "team")
  private List<Member> members = new ArrayList<>();
}

```

- Insert를 잘못하는 경우 
  - mappedBy 객체는 읽기 전용이기 때문에 값이 저장될 수 없음. 
  - 연관관계의 주인에 값을 입력하지 않음 

```java

public class JPAMain {
    public static void main(String[] args) {
        Member member = new Member();
        member.setUsername("Lines Bong");
        entityManager.persist(member);

        // Team을 저장하는 방법
        Team team = new Team();
        team.setName("TEAM A");
        team.getMembers().add(member);
        entityManager.persist(team); // 영속 상태가 되면 무조건 pk 값은 생성됨.

        entityTransaction.commit();
    }
}

```

- Insert가 제대로 되는 경우 

```java

public class JPAMain {
    public static void main(String[] args) {
    
        // Team을 저장하는 방법
        Team team = new Team();
        team.setName("TEAM A");
        entityManager.persist(team); // 영속 상태가 되면 무조건 pk 값은 생성됨.

        Member member = new Member();
        member.setUsername("Lines Bong");
        member.setTeam(team);
        entityManager.persist(member);
        
        entityTransaction.commit();
    }
}
```

- 양방향 매핑시 연관관계의 주인에 값을 입력해야한다 
  - 순수한 객체 관계를 고려하면 항상 양쪽다 값을 입력해야한다.
  - 이유는 영속성 컨택스트의 개념으로 인하여 만약 entityManager의 flush(), clear()가 실행되지 않는 상태라면, 1차 캐시에 값이 존재하지 않는다. 
    - 왜냐면 mappedBy가 선언된 객체에서 값이 지정되지 않았기 때문에 영속성 컨텍스트가 초기화 되기 전까지 메모리에 값이 없기 때문임 
  - 테스트 코드 작성시 JPA 없이 사용하는 경우, 
    - 코드 상의 이슈 , 에러를 발생시킬 수 있기 때문임. 

- 순수 객체 상태를 고려해서 항상 양쪽에 값을 설정하자 
- **연관관계 편의 메소드를 생성하자 : changeTeam(Team team) 부분 참고!**
  - 해당 시점에 선언하는 team에 대해서는 getter/setter보다는 명시적으로 함수를 정의하여 사용한다. 
  - lombok 등에 의한 자동화로 setter가 가지는 의미가 중요하지 않을 수 있기 때문에 개념적으로 분리한다. 
  - 연관관계 편의 메서드는 사용에 따라서 Member에 정의하거나, Team에 정의할 수 있는데, 양쪽에 존재하는 경우에는 문제가 있을 수 있음 
    - 따라서 한쪽에 정의하였을 경우에는 다른 한쪽은 제거 

```java

@Entity
public class Member {

    @Id
    @GeneratedValue
    @Column(name="MEMBER_ID")
    private Long id;

    @Column(name="USERNAME")
    private String username;

    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;

    public Team getTeam() {
        return team;
    }

    public void changeTeam(Team team) {
        this.team = team;
        team.getMembers().add(this);
    }
}

```

- Entity를 바로 Controller로 반환하는 경우,
  - 많은 문제가 생길 수 있음. 
    - JSON 생성 라이브러리를 사용하는 경우, 
      - 컨트롤러에는 Entity를 절대 반환하지 말 것! 절때 반환하지 말 것! 
      - Entity를 DTO로 변환하여 반환하라! 


## **주의사항**

- 운영 장비에는 절대 create, create-drp, update 사용하면안된다.
- 개발 초기 단계는 create 또는 update
- 테스트 서버는 update 또는 validate
- 스테이징과 운영 서버는 validate or none
