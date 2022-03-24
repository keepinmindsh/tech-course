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