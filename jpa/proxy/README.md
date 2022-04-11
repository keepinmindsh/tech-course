### 프록시와 연관관계 정리

- Member를 조회할 때 Team도 함께 조회해야 할 까?
    - 지연로딩이 필요한 이유

```java
public class JPAMain {
    public static void main(String[] args) {
        EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("hello");

        EntityManager entityManager = entityManagerFactory.createEntityManager();

        EntityTransaction entityTransaction = entityManager.getTransaction();

        entityTransaction.begin();

        try{

            Member member = entityManager.find(Member.class, 1L);
            printMemberAndTeam(member);

            entityTransaction.commit();
        }catch (Exception exception){
            entityTransaction.rollback();
        }finally {
            entityManager.close();
        }

        entityManagerFactory.close();
    }

    // 업무 로직에 따라서 Team을 가져와야하는 경우도 있고, Member만 가져와야하는 경우가 있을 때,
    // 만약 Member만 사용하고 Team을 사용할 일이 없는 경우, Team 객체는 불필요한 리소스 낭비가 잃어남. 
    private static void printMemberAndTeam(Member member) {

        String userName = member.getUsername();
        System.out.println("userName = " + userName);

        Team team = member.getTeam();
        System.out.println("team = " + team);
    }
}
```

- em.find(), em.getReference()
    - em.getReference(): 데이터 베이스를 조회를 미루는 가짜(프록시) 엔티티 객체 조회
    - 프록시
        - 실제 클래스를 상속 받아서 만들어짐
        - 실제 클래스와 겉 모양이 같다
        - 사용하는 입장에서는 진짜 객체인지 프록시 객체인지 구분하지 않고 사용하면됨 ( 이론상 )
        - 프록시 객체는 실제 객체의 참조를 보관
        - 프록시 객체를 호출하면 프록시 객체는 실제 객체의 메소드 호출
    - 프록시의 특징
        - 프록시 객체는 처음 사용할 때 한 번만 초기화
        - 프록시 객체를 초기화 할 때, 프록시 객체가 실제 엔티티로 바뀌는 것은 아님. 초기화 되면 프록시 객체를 통해서 실제 엔티티에 접근 가능
        - 프록시 객체는 원본 엔티티를 상속 받음, 따라서 타입 체크시 주의해야함 ( == 비교 실패, 대신 instance of 사용)
        - 영속성 컨텍스트에 찾는 엔티티가 이미 있으면 em.getReference()를 호출해도 실제 엔티티 반환
        - 영속성 컨텍스트의 도움을 받을 수 없는 준영속 상태일 때, 프록시를 초기화하는 문제 발생


```java
public class JPAMain {
  public static void main(String[] args) {
    try{

      Member member = new Member();
      member.setUsername("Id");

      entityManager.persist(member);

      entityManager.flush();
      entityManager.clear();

      //Member findMember = entityManager.find(Member.class, 1L);
      //printMemberAndTeam(findMember);

      // Proxy 객체 
      Member findRefMember = entityManager.getReference(Member.class, member.getId());
      System.out.println("findRefMember.getClass() = " + findRefMember.getClass());
      System.out.println("findRefMember.getUsername() = " + findRefMember.getUsername());

      entityTransaction.commit();
    }catch (Exception exception){
      entityTransaction.rollback();
    }finally {
      entityManager.close();
    }
  }
}

```

- 위의 코드 실행 결과!

```shell

findRefMember.getClass() = class bong.lines.jpashoping.proxy.Member$HibernateProxy$MweIMYII
findRefMember.getUsername() = Id


```