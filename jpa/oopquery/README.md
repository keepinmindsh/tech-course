# 객체지향 쿼리 언어
- JPQL
- JPA Criteria
- QueryDSL
- 네이티브 SQL
- JDBC API 직접 사용 , MyBatis , Spring JDBC Template 함께 사용

##### JPQL

- 가장 단순한 조회 방법
    - EntityManager.find()
    - 만약 복합적인 조회조건이 필요하다면?
- JPQL
    - JPA를 사용하는 엔티티 객체를 중심으로 개발
    - 문제는 검색 쿼리
    - 검색을 할 때도 테이블이 아닌 엔티티 객체를 대상으로 검색 해야함.
    - 모든 DB 데이터를 객체로 변환해서 검색하는 것은 불가능
    - 애플리케이션에서 필요한 데이터만 DB에서 불려오려면 결국 검색 조건이 포함된 SQL이 필요
    - JPA를 SQL를 추상화한 JPQL이라는 객체 지향 쿼리 언어 제공
    - SQL과 문법 유사 - SELECT, FROM, WHERE, GROUP BY, HAVING, JOIN 지원
    - JPQL는 엔티티 객체를 대상으로 분리
    - SQL은 데이터베이스 테이블을 대상으로 분리

```java
public class JPAMain {
  public static void main(String[] args) {
    // 실제 Entity를 대상으로 쿼리를 한 것임.   
    List<Member> result = entityManager.createQuery(
            "select m from Member m where m.userName like '%kim%'"
    ).getResultList();
  }
}

```

```shell

Hibernate: 
    /* select
        m 
    from
        Member m 
    where
        m.userName like '%kim%' */ select
            m1_0.MEMBER_ID,
            m1_0.USERNAME 
        from
            Member as m1_0 
        where
            m1_0.USERNAME like '%kim%'

```

- 테이블이 아닌 객체를 대상으로 검색하는 객체지향 쿼리
- SQL을 추상화해서 특정 데이터베이스에 SQL을 의존 X
- JPQL을 한마디로 정의하면 객체지향 SQL

##### Criteria 소개

- JPQL은 문자이기 때문에 동적 쿼리를 만드는 것이 어려움.
    - 동적쿼리를 작성하기 수월함.
    - 문자가 아닌 자바 코드로 JPQL을 작성할 수 있음.
    - 쿼리를 작성할 때 Compile 시점에 확인 할 수 있음!
- 실무에서 사용하기에는 어려운 부분이 있음. 코드 복잡성이 너무 올라감.
    - **Criteria 대신 QueryDSL 사용을 권장함.**

```java

public class JPAMain {
    public static void main(String[] args) {
        EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("hello");

        EntityManager entityManager = entityManagerFactory.createEntityManager();

        EntityTransaction entityTransaction = entityManager.getTransaction();

        entityTransaction.begin();

        try{
            CriteriaBuilder criteriaBuilder = entityManager.getCriteriaBuilder();

            CriteriaQuery<Member> criteriaQuery = criteriaBuilder.createQuery(Member.class);

            Root<Member> memberRoot = criteriaQuery.from(Member.class);

            criteriaQuery.select(memberRoot)
                            .where(criteriaBuilder.equal(memberRoot.get("username"),"kim"));

            entityManager.createQuery(criteriaQuery);

            entityTransaction.commit();
        }catch (Exception exception){
            exception.printStackTrace();
            entityTransaction.rollback();
        }finally {
            entityManager.close();
        }

        entityManagerFactory.close();
    }
}
```

```shell

Hibernate: 
    
    drop table if exists Member cascade 
Hibernate: 
    
    drop sequence if exists Member_SEQ
Hibernate: create sequence Member_SEQ start with 1 increment by 50
Hibernate: 
    
    create table Member (
       MEMBER_ID bigint not null,
        USERNAME varchar(255),
        primary key (MEMBER_ID)
    )
```

##### QueryDSL

- 문자가 아닌 자바 코드로 JPQL을 작성할 수 있음
- JPQL 빌더 역할
- 컴파일 시점에 문법 오류를 찾을 수 있음
- 동적 쿼리 작성 편리함.
- 단순하고 쉬움
- **실무 사용 권장**

```java

class Sample {
  public static void main(String[] args) {
      JPAFactoryQuery query = new JPAQueryFactory(em);
      QMember member = QMember.member;
      
      List<Member> list = 
              query.selectFrom(m)
                      .where(m.age.at(18))
                      .orderby(m.name.desc())
                      .fetch();
  }
}

```

##### 네이티브 SQL 소개

- JPA가 제공하는 SQL을 직접 사용하는 기능
- JPQL로 해결할 수 없는 특정 데이터베이스에 의존적인 기능
- 예) 오라클 CONNECT BY, 특정 DB만 사용하는 SQL 힌트

```java
public class JPAMain {
    public static void main(String[] args) {
        EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("hello");

        EntityManager entityManager = entityManagerFactory.createEntityManager();

        EntityTransaction entityTransaction = entityManager.getTransaction();

        entityTransaction.begin();

        try{

            entityManager.createNativeQuery("select MEMBER_ID, city, street, zipcode from MEMBER").getResultList();

            entityTransaction.commit();
        }catch (Exception exception){
            exception.printStackTrace();
            entityTransaction.rollback();
        }finally {
            entityManager.close();
        }

        entityManagerFactory.close();
    }
}
```

##### JDBC 직접 사용, SpringJDBCTemplate 조합 사용 가능

- JPA를 사용하면서 JDBC 커넥션을 직접 사용하거나, 스프링 JdbcTemplate, 마이바티스 등과 함께 사용 가능
- 단 영속성 컨텍스트를 적절한 시점에 강제로 플러시 필요
- 예) JPA를 우회해서 SQL를 실행하기 직전에 영속성 컨텍스트 수동 플러시
    - flush는 Query가 호출되거나 Commit되는 시점에 실행하기 때문에 nativeQuery 에서도 동작하지만, DB Connection을 직접 얻어와서
      사용하거나 JDBCTemplate, MyBatis 등을 사용할 때는 사전에 flush()를 실행해쥐야함. 왜냐면 flush는 jpa와 연관되어 있기 때문임

> [Query DSL](http://querydsl.com/)

### JPQL 소개

- JPQL는 객체지향 쿼리 언어다. 따라서 테이블을 대상으로 쿼리하는 것이 아니라 엔티티 객체를 대상으로 쿼리한다.
- **JPQL은 SQL을 추상화해서 특정 데이터베이스 SQL에 의존하지 않는다.**
- JPQL는 결국 SQL로 변환된다.

- JPQL 문법
    - select_절
        - from_절
        - [where_절]
        - [groupby_절]
        - [having_절]
        - [orderby_절]
    - update_문 :: = update_절 [where_절]
    - delete_문 :: = delete_절 [where_절]

- JPQL 기본 문법
    - select m from Member a m where m.age > 18
    - 엔티티와 속성은 대소문자 구분 O
    - JPQL 키워드는 대소문자 구분 x ( SELECT, FROM, where )
    - 엔티티 이름 사용, 테이블 이름이 아님
    - 별칭을 필수(m) (as는 생략가능)
- 집합과 정렬 함수 제공


- TypeQuery, Query
    - TypeQuery : 반한 타입이 명확할 때 사용
    - Query : 반환 타입이 명확하지 않을 때 사용

```java
public class JPAMain {
  public static void main(String[] args) {
    Member member = new Member();
    member.setUsername("member1");
    member.setAge(10);
    entityManager.persist(member);

    // 반환 타입이 명확한 경우
    TypedQuery<Member> memberTypedQuery1 = entityManager.createQuery("select m from Member m", Member.class);

    // 반환 타입이 문자열/숫자 등이 섞여 명확하지 않은 경우
    //Query query = entityManager.createQuery("select m.username from Member m");

    memberTypedQuery1.getResultList();
  }
}
```

- 결과 조회 API
    - query.getResultList() : 결과가 하나 이상일 때, 리스트 반환
        - 결과가 없으면 빈 리스트 반환
    - query.getSingleResult() : 결과가 정확히 하나, 단일 객체 반환
        - 결과가 정확히 하나일 경우에만 사용해야함.
            - 결과가 없으면 : javax.persistence.NoResultException
            - 둘 이상이면 : javax.persistence.NoUniqueResultException

```java
public class JPAMain {
  public static void main(String[] args) {
    // 반환 타입이 명확한 경우
    TypedQuery<Member> memberTypedQuery1 = entityManager.createQuery("select m from Member m", Member.class);

    // 반환 타입이 문자열/숫자 등이 섞여 명확하지 않은 경우
    //Query query = entityManager.createQuery("select m.username from Member m");

    List<Member> memberList = memberTypedQuery1.getResultList();

    for (Member memberItem : memberList) {
      System.out.println("memberItem.getUsername() = " + memberItem.getUsername());
    }
  }
}
```

- 파라미터 바인딩 - 이름 기준, 위치 기준
    - 가능하면 위치보다는 이름을 사용할 것 