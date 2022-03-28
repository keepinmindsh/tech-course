## **Persistence Context**

- 객체와 관계형 데이터베이스 매핑하기
    
![JPA Structure](https://github.com/keepinmindsh/tech-course/blob/main/assets/jpa_sturucture.png)

- 영속성 컨텍스트
    - JPA를 이해하는데 가장 중요한 용어
    - "엔티티를 영구 저장하는 환경"이라는 뜻
    - 논리적인 개념 : 눈에 보이지 않음
    - Entity Manager를 통해 영속성 컨텍스트에 접근

- 영속성 컨텍스트의 중요개념 
  - 1차 캐시
      - Entity 조회 시점에 생성된 영속 객체는 1차 캐시의 역할을 한다. 
      - 객체를 조회할 때, 영속성 컨텍스트에서 1차 캐시를 질의하여 데이터가 존재한다면 캐시에 있는 값을 그대로 조회함
      - 하나의 Transaction 안에서만 동작하기 때문에 효율성의 이점이 크지 않음. 다만 비즈니스 로직이 복잡할 경우는 도움이 될 수 있음
      - 객체에 대해서 저장하는 persist 시점에도 1차 캐시로 등록이 가능하다.
  - 동일성 보장
      - 1차 캐시로 반복 가능한 읽기(REPEATABLE READ) 등급의 트랜잭션 격리 수준을 데이터 베이스가 아닌 애플리케이션 차원에서 제공
        - Transaction Isolation 
          - READ UNCOMMITTED : 각 트랜잭션에서의 변경 내용이 COMMIT 이나 ROLLBACK 여부에 상관 없이 다른 트랜잭션에서 값을 읽을 수 있다.
          - READ COMMITTED : RDB 에서 대부분 기본적으로 사용되고 있는 격리 수준이다.
          - REPEATABLE READ : 하나의 트랜잭션내에서 똑같은 SELECT 쿼리를 실행했을 때는 항상 같은 결과를 가져와야 하는 REPEATABLE READ의 격리수준
          - SERIALIZABLE : 가장 단순한 격리 수준이지만 가장 엄격한 격리 수준
  - 트랜잭션을 지원하는 쓰기 지연
      - persist 시점에 데이터를 DB Insert 처리하지 않고 Commit 시점에 처리된다.
      - persist 를 다수 호출 후에 commit 명령어에 의해서 실행되므로 jdbc batch insert 와 동일하게 동작한다.
          - buffering 을 모아서 적용 가능
  - 변경 감지 ( Dirty Checking )
      - 영속 컨텍스트에 의해서 Transaction Commit (flush) 시점에 entity와 snapshot를 비교하는데,
        entity와 snapshot을 비교하여 변경된 값에 대해서 update query를 생성하여 commit 시점에 database에 반영된다.
  - 지연 로딩 ( Lazy Loading ) : Proxy

### 1차 캐시, 동일성 보장

```java
package bong.lines.persistenccontext;

import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.EntityTransaction;
import javax.persistence.Persistence;

public class JPAMain {
    public static void main(String[] args) {
        EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("hello");

        EntityManager entityManager = entityManagerFactory.createEntityManager();

        EntityTransaction entityTransaction = entityManager.getTransaction();

        entityTransaction.begin();

        try{

            Member member = new Member();
            member.setUserName("SHJeong");

            entityManager.persist(member);

            // Entity Manager 의 변경사항 적용
            entityManager.flush();
            // 1차 캐쉬 초기화
            entityManager.clear();

            Member member1 = entityManager.find(Member.class, 1L);

            Member member2 = entityManager.find(Member.class, 1L);

            System.out.println("member1 = " + member1);
            System.out.println("member2 = " + member2);

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

### JDBC Batch Insert ( JDBC 방식 )

```java

class Sample {
  public static void main(String[] args) {
    PreparedStatement ps = c.prepareStatement("INSERT INTO employees VALUES (?, ?)");

    ps.setString(1, "John");
    ps.setString(2,"Doe");
    ps.addBatch();

    ps.clearParameters();
    ps.setString(1, "Dave");
    ps.setString(2,"Smith");
    ps.addBatch();

    ps.clearParameters();
    
    int[] results = ps.executeBatch();
  }
}

```

### Entity Life Cycle

![Entity Lifecycle]()


### 참조 ( 이미지 )

> [merge for re attaching detached entities](https://javabydeveloper.com/merge-re-attaching-detached-entities/)
