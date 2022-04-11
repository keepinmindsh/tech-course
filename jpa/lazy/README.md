
### 즉시 로딩과 지연로딩

- 지연로딩 Lazy를 제공함.
    - 아래의 코드에서 Member내의 Team 객체에서 값을 가져오는 시점에 실제 쿼리가 수행되어 데이터를 가져옴.
    - 만약 Member, Team을 자주 함께 사용하는 경우에 EAGER를 이용해서 함께 조회 가능


- Lazy

```java

@Entity
public class Member {

    @Id
    @GeneratedValue
    @Column(name="MEMBER_ID")
    private Long id;

    @Column(name="USERNAME")
    private String username;

    public Team getTeam() {
        return team;
    }

    // 해당 코드는 읽기 전용이 됨. Insert, Update는 동작하지 않음 - 일대다 양방향 관계
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "TEAM_ID", insertable = false, updatable = false)
    private Team team;
}


```


- EAGER : Proxy가 아닌 진짜 객체를 바로 가져온다.
    - JPA 구현체는 가능하면 조인을 사용해서 SQL 한번에 함께 조회 하고자 함.

```java

@Entity
public class Member {

    @Id
    @GeneratedValue
    @Column(name="MEMBER_ID")
    private Long id;

    @Column(name="USERNAME")
    private String username;

    public Team getTeam() {
        return team;
    }

    // 해당 코드는 읽기 전용이 됨. Insert, Update는 동작하지 않음 - 일대다 양방향 관계
    @ManyToOne(fetch = FetchType.EAGER)
    @JoinColumn(name = "TEAM_ID", insertable = false, updatable = false)
    private Team team;
}

```

- 프록시와 즉시 로딩 주의
    - **가급적 지연 로딩만 사용 ( 특히 실무에서 )**
    - 즉시 로딩을 적용하면 예상하지 못한 SQL이 발생
        - 개발자가 예상하지 못한 Join Query가 생성되어 나가게됨.
    - 즉시 로딩은 JPQL에서 N + 1 문제를 일으킨다.
        - 동적으로 원하는 쿼리를 작성할 때 ( FethJoin) 에 의해서 N+1 쿼리를 제거하여 사용할 수 있다.
        - 쿼리가 중복으로 호출되는 케이스
    - @ManyToOne, @OneToOne은 기본이 즉시 로딩 -> LAZY로 설정
    - @OneToMany, @ManyToMany 는 기본이 지연로딩

- 지연 로딩 활용 > 이론적인 내용으로 실무에서는 다 지연로딩으로 구성해야함.
    - Member와 Team은 자주 함께 사용 - 즉시 로딩
    - Member와 Order는 가끔 사용 - 지연 로딩
    - Order와 Product는 자주 함께 사용
- 지연 로딩 활용 - 실무
    - 모든 연관관계에서 지연 로딩을 사용해라
    - 실무에서 즉시 로딩을 사용하지 마라!
    - JPQL fetch 조인이나, 엔티티 그래프 기능을 사용해라.
    - 즉시 로딩은 상상하지 못한 쿼리가 나간다. 