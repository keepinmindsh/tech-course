### 데이터 중심 설계의 문제점

- 현재 방식은 객체 설계를 테이블 설계에 맞춘 방식
- 테이블의 외래키를 객체에 그대로 가져옴
- 객체 그래프 탐색이 불가능
- 참조가 없으므로 UML도 잘못됨.

### 연관관계 매핑 기초

- 객체와 테이블 연관관계의 차이를 이해
- 객체의 참조와 테이블의 외래키를 매핑
    - 방향 : 단방향, 양방향
    - 다중성 : 다대일, 일대다, 일대일, 다대다
    - 연관관계의 주인 : 객체 양방향 연관관계는 관리 주인이 필요

- 객체 지향 설계의 목표는 객체들의 협력 공동체를 만드는 것이다.
- 객체를 테이블에 맞추어 모델링

### 객체를 테이블에 맞추어 데이터 중심으로 모델링하면, 협력 관계를 만들수 없다.

- 테이블은 외래키로 조인을 사용해서 연관된 테이블을 찾는다.
- 객체는 참조를 사용해서 연관된 객체를 찾는다.
- 테이블과 객체 사이에 이런 큰 간격이 있다. 


## 단방향 연관 관계

- JPA 에서의 객체지향 모델링의 방식은 아래와 같다.

```java

package bong.lines.jpashoping.step2;

import javax.persistence.*;

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

    public void setTeam(Team team) {
        this.team = team;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }
}

```

```java

public class JPAMain {
    public static void main(String[] args) {
        EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("hello");

        EntityManager entityManager = entityManagerFactory.createEntityManager();

        EntityTransaction entityTransaction = entityManager.getTransaction();

        entityTransaction.begin();

        try{

            Team team = new Team();
            team.setName("TEAM A");
            entityManager.persist(team);

            Member member = new Member();
            member.setUsername("Lines Bong");
            member.setTeam(team);

            entityManager.persist(member);

            // 만약 영속성 컨텍스트가 아닌 db에서 쿼리로 조회해오고 싶다면,
            entityManager.flush(); // db에 모든 값 반영 - commit 전에 반영 가능
            entityManager.clear(); // 영속성 컨텍스트 초기화

            Member member1 = entityManager.find(Member.class, member.getId());

            Team team1 = member.getTeam();

            System.out.println("team1.getTeamId() = " + team1.getTeamId());
            System.out.println("team1.getName() = " + team1.getName());

            entityTransaction.commit();
        }catch (Exception exception){
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
    select
        m1_0.MEMBER_ID,
        t1_0.TEAM_ID,
        t1_0.name,
        m1_0.USERNAME 
    from
        Member as m1_0 
    left outer join
        Team as t1_0 
            on m1_0.TEAM_ID = t1_0.TEAM_ID 
    where
        m1_0.MEMBER_ID = ?
team1.getTeamId() = 1
team1.getName() = TEAM A

```

## 양방향 연관 관계와 연관관계의 주인

#### 양방향 매핑
  - 양쪽으로 참조해서 값을 가질 수 있을 경우
    - 객체의 참조와 테이블의 외래키의 차이

#### 연관관계의 주인과 mappedBy
  - 객체와 테이블 간에 연관관계를 맺는 차이를 이해한다.
    - 객체 연관관계 2개
      - 회원 -> 팀 연관관계 1개 ( 단방향 )
      - 팀 -> 회원 연관관계 1개 ( 단방향 )
    - 테이블 연관관계 = 1 개
      - 회원 <-> 팀의 연관관계 1개 ( 양방향 )

#### 객체의 양방향 관계
  - 객체의 양방향 관계는 사실 양방향 관계가 아니라 서로 다른 단방향 관계 2개다.
  - 객체를 양방향으로 참조하려면 단방향 연관관계 2개를 만들어야 한다.

#### 테이블의 양방향 관계
  - 테이블은 외래키 하나로 두 테이블의 연관관계를 관리
  - MEMBER.TEAM_ID 외래키 하나로 양방향 연관관계 가짐 ( 양쪽으로 조인할 수 있다. )


#### 딜레마를 해결하는 방법
- 딜레마 : Team 객체와 Member 객체에 존재하는 teamId를 어느 곳에서든 업데이트 할 수 있는데.. 어디서 해야하지?
- Member와 Team의 객체 내에 존재하는 member의 값을 어떻게 관리해야 하는가?
  - 둘 중 하나로 외래키를 관리해야 한다.

#### 연관관계의 주인
- 양방향 매핑 규칙
  - 객체의 두 관계 중 하나를 연관관계의 주인으로 지정
  - **연관관계의 주인만이 외래키를 관리 ( 등록, 수정 )**
  - **주인이 아닌 쪽은 읽기만 가능**
  - 주인은 mappedBy 속성 사용 불가
  - 주인이 아닌면 mappedBy 속성으로 주인 지정
    - mappedBy의 속성의 값에 대해서는 값을 수정하는 것은 변경 불가

#### 그렇다면 누구를 주인으로?
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
        EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("hello");

        EntityManager entityManager = entityManagerFactory.createEntityManager();

        EntityTransaction entityTransaction = entityManager.getTransaction();

        entityTransaction.begin();

        try{

            Member member = new Member();
            member.setUsername("Lines Bong");
            entityManager.persist(member);

            // Team을 저장하는 방법
            Team team = new Team();
            team.setName("TEAM A");
            team.getMembers().add(member);
            entityManager.persist(team); // 영속 상태가 되면 무조건 pk 값은 생성됨.


            // 만약 영속성 컨텍스트가 아닌 db에서 쿼리로 조회해오고 싶다면,
            entityManager.flush(); // db에 모든 값 반영 - commit 전에 반영 가능
            entityManager.clear(); // 영속성 컨텍스트 초기화


            entityTransaction.commit();
        }catch (Exception exception){
            entityTransaction.rollback();
        }finally {
            entityManager.close();
        }

        entityManagerFactory.close();
    }
}

```

- Insert가 제대로 되는 경우

```java

public class JPAMain {
    public static void main(String[] args) {
        EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("hello");

        EntityManager entityManager = entityManagerFactory.createEntityManager();

        EntityTransaction entityTransaction = entityManager.getTransaction();

        entityTransaction.begin();

        try{

            // Team을 저장하는 방법
            Team team = new Team();
            team.setName("TEAM A");
            entityManager.persist(team); // 영속 상태가 되면 무조건 pk 값은 생성됨.

            Member member = new Member();
            member.setUsername("Lines Bong");
            member.setTeam(team);
            entityManager.persist(member);

            // 만약 영속성 컨텍스트가 아닌 db에서 쿼리로 조회해오고 싶다면,
            entityManager.flush(); // db에 모든 값 반영 - commit 전에 반영 가능
            entityManager.clear(); // 영속성 컨텍스트 초기화

            entityTransaction.commit();
        }catch (Exception exception){
            entityTransaction.rollback();
        }finally {
            entityManager.close();
        }

        entityManagerFactory.close();
    }
}
```

#### 양방향 매핑시 연관관계의 주인에 값을 입력해야한다
- 순수한 객체 관계를 고려하면 항상 양쪽다 값을 입력해야한다.
- 이유는 영속성 컨택스트의 개념으로 인하여 만약 entityManager의 flush(), clear()가 실행되지 않는 상태라면, 1차 캐시에 값이 존재하지 않는다.
  - 왜냐면 mappedBy가 선언된 객체에서 값이 지정되지 않았기 때문에 영속성 컨텍스트가 초기화 되기 전까지 메모리에 값이 없기 때문임
- 테스트 코드 작성시 JPA 없이 사용하는 경우,
  - 코드 상의 이슈 , 에러를 발생시킬 수 있기 때문임.

```java

public class JPAMain {
  public static void main(String[] args) {
    EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("hello");

    EntityManager entityManager = entityManagerFactory.createEntityManager();

    EntityTransaction entityTransaction = entityManager.getTransaction();

    entityTransaction.begin();

    try {

      // Team을 저장하는 방법
      Team team = new Team();
      team.setName("TEAM A");
      entityManager.persist(team); // 영속 상태가 되면 무조건 pk 값은 생성됨.

      Member member = new Member();
      member.setUsername("Lines Bong");
      member.setTeam(team);

      team.getMembers().add(member);

      entityManager.persist(member);

      // 만약 영속성 컨텍스트가 아닌 db에서 쿼리로 조회해오고 싶다면,
      entityManager.flush(); // db에 모든 값 반영 - commit 전에 반영 가능
      entityManager.clear(); // 영속성 컨텍스트 초기화

      entityTransaction.commit();
    } catch (Exception exception) {
      entityTransaction.rollback();
    } finally {
      entityManager.close();
    }

    entityManagerFactory.close();
  }
}

```

#### 순수 객체 상태를 고려해서 항상 양쪽에 값을 설정하자
- **연관관계 편의 메소드를 생성하자 : changeTeam(Team team) 부분 참고!**
  - 해당 시점에 선언하는 team에 대해서는 getter/setter보다는 명시적으로 함수를 정의하여 사용한다.
  - lombok 등에 의한 자동화로 setter가 가지는 의미가 중요하지 않을 수 있기 때문에 개념적으로 분리한다.
  - 연관관계 편의 메서드는 사용에 따라서 Member에 정의하거나, Team에 정의할 수 있는데, 양쪽에 존재하는 경우에는 문제가 있을 수 있음
    - 따라서 한쪽에 정의하였을 경우에는 다른 한쪽은 제거

```java

// Member에 연관관계 편의 메서드를 제공하는 경우 

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

```java

// Team에 연관관계 편의 메서드를 제공하는 경우 

@Entity
@SequenceGenerator(name="team_seq_generator", sequenceName = "team_seq", allocationSize = 1)
public class Team {

    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "team_seq_generator")
    @Column(name="TEAM_ID")
    private Long teamId;

    private String name;

    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<>();

    public void addMember(Member member){
        member.setTeam(this);
        members.add(member);
    }

    public Long getTeamId() {
        return teamId;
    }

    public void setTeamId(Long teamId) {
        this.teamId = teamId;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public List<Member> getMembers() {
        return members;
    }

    public void setMembers(List<Member> members) {
        this.members = members;
    }
}
```

#### 양방향 매핑시 무한 루프를 조심하자
  - 예) toString(), lombok, JSON 생성 라이브러리
    - toString() 의 무한 루프 사례

```java

public class Team {
    @Override
    public String toString(){
        return "Team (" +
                "id=" + id +
                ", name=" + name + '\'' +
                ", members=" + members + 
                ")";
    }
}

```

####  Entity를 바로 Controller로 반환하는 경우,
  - 많은 문제가 생길 수 있음.
    - JSON 생성 라이브러리를 사용하는 경우,
      - 컨트롤러에는 Entity를 절대 반환하지 말 것! 절때 반환하지 말 것!
      - Entity를 DTO로 변환하여 반환하라!

#### **양방향 매핑 정리**
  - 단방향 매핑 만으로도 이미 연관관계 매핑은 완료
  - 양방향 매핑은 반대방향으로 조회 ( 객체 그래프 탐색 ) 기능이 추가된 것 뿐
  - JPQL에서 역방향 탐색할 일이 많음
  - 단방향 매핑을 잘하고 양방향 은 필요할 때 추가해도됨. ( 테이블에 영향을 주지 않음 )
- **연관관계의 주인을 정하는 기준**
  - 연관관계의 주인은 외래키의 위치를 기준으로 정리해야함.
  - 비즈니스 로직을 기준으로 연관관계의 주인을 선택하면 안됨.

#### 다양한 연관 관계의 매핑 정리 
- 연관관계 매핑시 고려 사항
  - 단방향, 양방향
    - 테이블
      - 외래키 하나로 양쪽 조인 가능
      - 사실 방향이라는 개념이 없음
    - 객체
      - 참조용 필드가 있는 쪽으로만 참조 가능
      - 한쪽만 참조하면 단방향
      - 양쪽이 서로 참조하면 양방향 ( 단방향이 2개인 것 - 서로 양쪽의 참조관계가 존재함 )
  - 연관관계의 주인
    - 테이블은 외래키 하나로 두 테이블이 연관관계를 맺음
    - 객체 양방향 관계는 A->B, B->A 처럼 참조가 2군데
    - 객체 양방향 관계넌 참조가 2군데 있음, 둘중 테이블의 외래키를 관리할 곳을 지정해야함
    - 연관관계의 주인: 외래키를 관리하는 참조
    - 주인의 반대편: 외래키에 영향을 주지 않음.

##### 다대일 ( N : 1 ) @ManyToOne

- 가장 많이 사용하는 연관관계
  - @ManyToOne을 사용한 객체를 기준으로 Primary Key를 가지는 테이블에 대해서 @OneToMany로 데이터를 가져올 수 있다.
    - MEMBER와 TEAM의 관계

##### 일대다 ( 1 : N ) @OneToMany

- 일대다 단방향은 일대다(1:N)에서 일(1)이 연관관계의 주인
- 테이블 일대다 관계는 항상 다(N) 쪽에 외래키가 있음
- 객체와 테이블의 차이 때문에 반대편 테이블의 외래키를 관리하는 특이한 구조
- @JoinColumn을 꼭 사용해야함. 그렇지 않으면 조인 테이블 방식을 사용함 ( 중간에 테이블을 하나 추가함 )
  - 테이블이 신규로 하나더 추가되기 때문에 성능상, 운영상 좋은 방식은 아님 ( 조인 테이블 )

```java
@Entity
public class Member {

    @Id
    @GeneratedValue
    @Column(name="MEMBER_ID")
    private Long id;

    @Column(name="USERNAME")
    private String username;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }
}

```

```java
@Entity
@SequenceGenerator(name="team_seq_generator", sequenceName = "team_seq", allocationSize = 1)
public class Team {

    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "team_seq_generator")
    @Column(name="TEAM_ID")
    private Long teamId;

    private String name;

    @OneToMany
    @JoinColumn(name = "TEAM_ID")
    private List<Member> members = new ArrayList<>();

    public Long getTeamId() {
        return teamId;
    }

    public void setTeamId(Long teamId) {
        this.teamId = teamId;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public List<Member> getMembers() {
        return members;
    }

    public void setMembers(List<Member> members) {
        this.members = members;
    }
}
```

- 1:N - OneToMany 연관관계 코드

```java

public class JPAMain {
    public static void main(String[] args) {
        EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("hello");

        EntityManager entityManager = entityManagerFactory.createEntityManager();

        EntityTransaction entityTransaction = entityManager.getTransaction();

        entityTransaction.begin();

        try{

            Member member = new Member();
            member.setUsername("bong1");

            entityManager.persist(member);

            Team team = new Team();

            team.setName("Bong Team");
            team.getMembers().add(member);

            entityManager.persist(team);

            entityTransaction.commit();
        }catch (Exception exception){
            entityTransaction.rollback();
        }finally {
            entityManager.close();
        }

        entityManagerFactory.close();
    }
}

```

- 실행 결과

```shell
Hibernate: 
    /* insert bong.lines.jpashoping.onetomany.Member
        */ insert 
        into
            Member
            (USERNAME, MEMBER_ID) 
        values
            (?, ?)
Hibernate: 
    /* insert bong.lines.jpashoping.onetomany.Team
        */ insert 
        into
            Team
            (name, TEAM_ID) 
         values
            (?, ?)
Hibernate: 
    /* create one-to-many row bong.lines.jpashoping.onetomany.Team.members */ update
        Member 
    set
        TEAM_ID=? 
    where
        MEMBER_ID=?
```

- 일대다 연관관계를 잘 사용하지 않는 이유
  - 자바 코드 상에서 로직과 테이블의 연관관계 상 혼란이 올 수 있음
  - 테이블이 수십개가 돌아가는 운영 기준에서는 혼란이 올 수 있음
  - 엔티티가 관리하는 외래키가 다른 테이블에 있음
  - 연관 관계 관리를 위해서 추가로 UPDATE SQL 실행
- **일대다 단방향보다는 다대일 양방향 매핑을 사용하자**


- 일대다 양방향 정리
  - 이런 매핑은 공식적으로 X
  - @JoinColumn(insertable=false, updatable=false)
  - 읽기 전용 필드를 사용해서 양방향 처럼 사용하는 방법
  - **다대일 양방향을 사용하자**

```java

@Entity
public class Member {

    @Id
    @GeneratedValue
    @Column(name="MEMBER_ID")
    private Long id;

    @Column(name="USERNAME")
    private String username;

    // 해당 코드는 읽기 전용이 됨. Insert, Update는 동작하지 않음 - 일대다 양방향 관계 
    @ManyToOne
    @JoinColumn(name = "TEAM_ID", insertable = false, updatable = false)
    private Team team;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }
}

```

##### 일대일 ( 1 : 1 ) @OneToOne

- 일대일 관계는 그 반대도 일대일
- 주 테이블이나 대상 테이블 중에 외래키 선택 가능
  - 주 테이블에 외래키
    - 다대일 단뱡향 매핑과 유사함.
    - 객체 지향 개발자 선호
    - JPA 매핑 정리
    - 장점 : 주테이블만 조회해도 대상 테이블에 데이터가 있는지 확인 가능
    - 단점 : 값이 없으면 외래키에 NULL 허용
  - 대상 테이블에 외래키
    - 대상 테이블에 외래키가 존재
    - 전통적인 데이터 베이스 개발자 선호
    - 장점 : 주테이블과 대상 테이블을 일대일에서 일대다 돤계로 변경할 때 테이블 구조 유지
    - 단점 : 프록시 기능의 한계로 지연로딩으로 설정해도 항상 즉시 로딩됨
- 외래키에 데이터베이스 유니크 제약 조건 추가
- 다대일 양방향 매핑 처럼 외래키가 있는 곳이 연관관계의 주인
- 반대편은 mappedBy 적용

```java
@Entity
public class Locker {

    @Id
    @GeneratedValue
    private Long id;

    private String name;

    @OneToOne(mappedBy = "locker")
    private Member member;
}
```

```java
@Entity
public class Member {

    @Id
    @GeneratedValue
    @Column(name="MEMBER_ID")
    private Long id;

    @Column(name="USERNAME")
    private String username;

    @OneToOne
    @JoinColumn(name = "LOCKER_ID")
    private Locker locker;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }
}
```

##### 다대다 ( N : M) @ManyToMany

- 실무에서는 사용하면 안됨
  - 관계형 데이터베이스는 정규화된 테이블 2개로 다대다 관계를 표현할 수 없음
  - 연결 테이블을 추가해서 일대다, 다대일 관계로 풀어내야함.
  - 객체는 컬렉션을 사용해서 객체 2개로 다대다 관계 가능
    - 테이블에서는 Join Table을 이용해서 관계를 구성이 가능함.

- 편리해보이지만 실무에서 사용 안함
- 연결 테이블이 연결만 하고 끝나지 않음
- 주문 시간, 수량 같은 데이터가 들어올 수 있음]

```java

@Entity
public class Member {

    @Id
    @GeneratedValue
    @Column(name="MEMBER_ID")
    private Long id;

    @Column(name="USERNAME")
    private String username;

    @ManyToMany
    @JoinTable(name = "MEMBER_PRODUCT")
    private List<Product> products = new ArrayList<>();
    
    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }
}

```

```java

@Entity
public class Product {

    @Id @GeneratedValue
    private Long id;


    @ManyToMany(mappedBy = "products")
    List<Member> members = new ArrayList<>();

}

```

```shell

Hibernate: 
    
    create table Member (
       MEMBER_ID bigint not null,
        USERNAME varchar(255),
        TEAM_ID bigint,
        primary key (MEMBER_ID)
    )
Hibernate: 
    
    create table MEMBER_PRODUCT (
       members_MEMBER_ID bigint not null,
        products_id bigint not null
    )
Hibernate: 
    
    create table Product (
       id bigint not null,
        primary key (id)
    )
Hibernate: 
    
    create table Team (
       TEAM_ID bigint not null,
        name varchar(255),
        primary key (TEAM_ID)
    )
Hibernate: 
    
    alter table if exists Member 
       add constraint FKl7wsny760hjy6x19kqnduasbm 
       foreign key (TEAM_ID) 
       references Team
Hibernate: 
    
    alter table if exists MEMBER_PRODUCT 
       add constraint FKc6hsxwm11n18ahnh5yvbj62cf 
       foreign key (products_id) 
       references Product
Hibernate: 
    
    alter table if exists MEMBER_PRODUCT 
       add constraint FKp9hlrsu8hrsusdymar0ddcl9o 
       foreign key (members_MEMBER_ID) 
       references Member


```

- 다대다 한계 극복
  - @ManyToMany -> @OneToMany, @ManyToOne
  - 중간 테이블(JoinTable)을 중간 Entity로 승격 처리( 연결 테이블을 엔티티 승격 )

```java

@Entity
public class Member {

    @Id
    @GeneratedValue
    @Column(name="MEMBER_ID")
    private Long id;

    @Column(name="USERNAME")
    private String username;

    @OneToMany(mappedBy = "member")
    private List<MemberProduct> memberProducts = new ArrayList<>();

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }
}

```

```java

@Entity
public class MemberProduct {

    @Id @GeneratedValue
    private Long id;

    @ManyToOne
    @JoinColumn(name = "MEMBER_ID")
    private Member member;

    @ManyToOne
    @JoinColumn(name= "PRODUCT_ID")
    private Product product;

}

```

```java

@Entity
public class Product {

    @Id @GeneratedValue
    private Long id;

    @OneToMany(mappedBy = "product")
    List<MemberProduct> memberProducts = new ArrayList<>();

}

```

### 연관관계 정리 하기

```java

@Entity
public class Member {
    @Id
    @Column(name="MEMBER_ID")
    @GeneratedValue
    private Long id;

    private String name;

    private String city;

    private String street;

    private String zipcode;

    private String orders;
}

@Entity
@Table(name = "ORDERS")
public class Order {

  @Id
  @Column(name = "ORDER_ID")
  @GeneratedValue
  private String id;

  @ManyToOne
  @JoinColumn(name = "MEMBER_ID")
  private Member member;

  @OneToOne
  @JoinColumn(name="DELIVERY_ID")
  private Delivery delivery;

  @OneToMany(mappedBy = "order")
  private List<OrderItem> orderItems = new ArrayList<>();

  private LocalDate orderDate;

  private OrderStatus orderStatus;

}

@Entity
public class OrderItem {

  @Id
  @Column(name="ORDER_ITEM_ID")
  @GeneratedValue
  private Long id;

  @ManyToOne
  @JoinColumn(name="ORDER_ID")
  private Order order;

  private int orderPrice;

  private int count;
}


@Entity
public class Delivery {

  @Id
  @Column(name = "DELIVERY_ID")
  @GeneratedValue
  private String id;

  @OneToOne(mappedBy = "delivery")
  private Order order;

  private String city;

  private String street;

  private String zipcode;

  private DeliveryStatus deliveryStatus;
}

@Entity
public class Item {

  @Id
  @Column(name = "ITEM_ID")
  @GeneratedValue
  private Long id;

  private String name;

  private int price;

  private int stockQuantity;

  @ManyToMany(mappedBy = "items")
  private List<Category> categories = new ArrayList<>();
}


@Entity
public class Category {

  @Id
  @GeneratedValue
  @Column(name = "CATEGORY_ID")
  private Long id;

  @ManyToMany
  @JoinTable(name = "CATEGORY_ITEM", joinColumns = @JoinColumn(name = "CATEGORY_ID"), inverseJoinColumns = @JoinColumn(name = "ITEM_ID"))
  private List<Item> items = new ArrayList<>();

  @ManyToOne
  @JoinColumn(name = "PARENT_IO")
  private Category parent;

  @OneToMany(mappedBy = "parent")
  private List<Category> child;
}

```

- @JoinColumn
  - name : 매핑할 외래키 이름
  - referencedColumnName : 외래키가 참조하는 대상 테이블의 컬럼명
  - foreignKey(DDL) : 외래키 제약 조건은 직접 지정할 수 잇다. 이 속성은 테이블 생성시만 사용 가능
  - unique, nullable, insertable, updatable, columnDefinition, table : @Column 속성과 같음.

- @ManyToOne - 연관관계의 주인이 되어야함.
  - optional : false로 설정하면 연관된 엔티티가 항상 있어야 한다.
  - fetch : 글로벌 패치 전략을 설정한다.
    - @ManyToOne : FetchType.EAGER
    - @OneToMany : FetchType.LAZY
  - cascade : 영속성 전이 기능을 사용한다.
  - targetEntity : 연관된 엔티티의 타입 정보를 설정한다. 이기능은 거의 사용하지 않는다. 컬렉션을 사용해도 제네릭으로 타입정보를 알 수 있다.

- @OneToMany
  - mappedBy : 연관관계의 주인을 선택한다.
  - fetch
  - cascade
  - targetEntity