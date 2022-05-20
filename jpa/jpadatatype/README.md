# JPA 데이터 타입 분류

- 엔티티 타입
    - @Entity로 정의하는 객체
    - 데이터가 변해도 식별자를 지속해서 추척 가능
    - 예) 회원 엔티티의 키나 나이 값을 변경해도 식별자로 인식 가능
- 값 타입
    - int, integer, String 처럼 단순히 값을 사용하는 자바 기본 타입이나 객체
    - 식별자가 없고 값만 있으므로 변경시 추적 불가
    - 예) 숫자 100을 200으로 변경하면 완전히 다른 값으로 대체

- 기본값 타입
    - 자바 기본 타입(int, double)
    - 래퍼 클래스(Integre, Long)
    - String
- 임베디드 타입
    - embedded type, 복합 값 타임
- 컬렉션 값 타입
    - collection value type


- 기본 값 타입
    - 예) String name, int age
    - 생명주기를 엔티티에 의존
        - 예) 회원을 삭제하면 이름, 나이 필드도 함께 삭제
    - 값 타입은 공유하면 안됨
        - 예) 회원 이름 변경시 다른 회원의 이름이 변경되어서는 안됨.

- 참고 : 자바의 기본 타입은 절대 공유 X
    - int, double 같은 기본 타입은 절대 공유 X
    - 기본 타입은 항상 값을 복사함.
    - integer 같은 래퍼 클래스나 String 같은 특수한 클래스는 공유 가능한 객체이지만 변경 X


- 임베디드 타입 ( 복합값 타입 )
    - 새로운 값 타입을 직접 정의할 수 있음
    - JPA는 임베디드 타입(embedded type)이라 함
    - 주로 기본 값을 모아서 만들어서 복합 값 타입이라고도 함.
    - int, String과 같은 값 타입

- 임베디드 타입 사용법
    - @Embeddable : 값 타입을 정의하는 곳에 표시
    - @Embedded : 값 타입을 사용하는 곳에 표시
    - 기본 생성자 필수

- 임베디드 타입의 장점
    - 재사용
    - 높은 응집도
    - Period.isWork() 처럼 해당 값 타입만 사용하는 의미 있는 메소드를 만들 수 있음
    - 임베디드 타입을 포함한 모든 값 타입은, 값 타입을 소유한 엔티티에 생명주기를 의존함

- 임베디드 타입 사용법
    - @Embeddable : 값 타입을 정의하는 곳에 표시
    - @Embedded : 값 타입을 사용하는 곳에 표시
    - 기본 생성자 필수

- 임베디드 타입과 테이블 매핑
    - 임베디드 타입은 엔티티 값일 뿐이다.
    - 임베디드 타입을 사용하기 전과 후에 매핑하는 테이블은 같다.
    - 객체와 테이블을 아주 세밀하게 매핑하는 것이 가능
    - 잘 설계한 ORM 애플리케이션은 매핑한 테이블의 수보다 클래스의 수가 더 많음

- @AttributeOverride : 속성 재정의
    - 한 엔티티에서 같은 값 타입을 사용하면?
    - 컬럼명이 중복됨.
    - @AttributeOverrides, @AttributeOverride를 사용해서 컬럼 명 속성을 재정의

- 임베디드 타입과 null
    - 임베디드 타입의 값이 null이면 매핑한 컬럼 값은 모두 null


```java


@Entity
public class Member {
    @Id
    @Column(name="MEMBER_ID")
    @GeneratedValue
    private Long id;

    @Column(name = "USERNAME")
    private String name;

    @Embedded
    private Period workPeriod;

    @Embedded
    private Address homeAddress;

    @Embedded
    @AttributeOverrides({
            @AttributeOverride(name="city", column=@Column(name = "WORK_CITY")),
            @AttributeOverride(name="street", column=@Column(name = "WORK_STREET")),
            @AttributeOverride(name="zipcode", column=@Column(name = "WORK_ZIPCODE")),
    })
    private Address workAddress;
}


```

- 값 타입과 불변 객체
    - 값 타입은 복잡한 객체 세상을 조금이라도 단순화하려고 만든 개념이다. 따라서 값 타입은 단순하고 안전하게 다룰 수 있어야 한다.

- 값 타입 공유 참조
    - 임베디드 타입 같은 값 타입을 여러 엔티티에서 공유하면 위험함
    - 부작용 발생
    - 값 타입 복사
        - 값 타입의 실제 인스턴스 값을 공유하는 것은 위험
        - 대신 값을 복사해서 사용

- 객체 타입의 한계
    - 항상 값을 복사해서 사용하면 공유 참조로 인해 발생하는 부작용을 피할 수 있다.
    - 문제는 임베디드 타입 처럼 직접 정의한 값 타입은 자바의 기본 타입이 아니라 객체 차입니다.
    - 자바 기본 타입에 값을 대입하면 값을 복사한다.
    - 객체 타입은 참조 값을 직접 대입하는 것을 막을 방법이 없다.
    - 객체의 공유 참조는 피할 수 없다.

이로 인해서 이를 해결하고자 하여 불변 객체를 사용한다.

- 불변 객체
    - 객체 타입을 수정할 수 없게 만들면 부작용을 원천 차단
    - 값 타입은 불변 객체로 설계해야함.
    - 불변 객체 : 생성 시점 이후 절대 값을 변경할 수 없는 객체
    - 생성자로만 값을 설정하고 수정자를 만들지 않으면 됨
        - setter를 사용하지 않음
    - 참고 : Integer, String은 자바가 제공하는 대표적인 불변 객체

불변 객체로 정의된 객체에 대해서 수정을 하고 싶은 경우에는 새로운 객체를 생성해서 저장해야함.

- 값 타입의 비교
    - 값 타입 : 인스턴스가 달라도 그 안에 값이 같으면 같은 것으로 봐야함.
    - 동일성 비교 : 인스턴스의 참조 값을 비교, == 사용
    - 동등성 비교 : 인스턴스의 값을 비교 , equals() 사용
    - 값 타입은 a.equals(b)를 사용해서 동등성을 비교를 해야함
    - 값 타입의 equals 메소드를 적절하게 재정의 (주로 모든 필드 사용)

- 값 타입 컬렉션
    - 값 타입 컬렉션은 영속성 전이 + 고아 객체 제거 기능을 필수로 가진다고 볼 수 있다.
    - 값 타입 컬렉션은 모두 지연 로딩 방식임
    - 값 타입을 하나 이상 저장할 때 사용
    - @ElementCollection, @CollectionTable 사용
    - 데이터 베이스는 컬렉션을 같은 테이블에 저장할 수 없다.
    - 컬렉션을 저장하기 위한 별도의 테이블이 필요함.


- 값 타입 컬렉션의 제약 사항
    - 값 타입은 엔티티와 다르게 식별자 개념이 없다.
    - 값은 변경하면 추적이 어렵다.
    - 값 타입 컬렉션에 변경 사항이 발생하면, 주인 엔티티와 연관된 모든 데이터를 삭제하고, 값 타입 컬렉션에 있는 현재 값을 모두 다시 저장한다.
    - 값 타입 컬렉션을 매핑하는 테이블은 모든 컬럼을 묶어서 기본 키를 구성해야함 : null 입력 x, 중복저장 x

- 값타입은 사용하지 않는 방향을 추천함.

- 실무에서는 값 타입 컬렉션 대신에 일대다 관계를 고려하는게 나음.

- 값 타입 컬렉션 대안
    - 실무에서는 상황에 따라 값 타입 컬렉션 대신에 일대다 관계를 고려
    - 일대다 관계를 위한 엔티티를 만들고, 여기에서 값 타입을 사용
    - 영속성 전이(Cascade) + 고아 객체 제거를 사용해서 값타입 컬렉션 처럼 사용

- 값 타입 컬렉션은 주로 단순한 멀티셀렉트 박스 상에서 사용할 경우 간단하게 사용가능하다.


- 엔티티 타입의 특징
    - 식별자 있음
    - 생명주기 관리
    - 공유
- 값 타입 특징
    - 식별자 X
    - 생명주기를 엔티티에 의존
    - 공유하지 않는 것이 안전 (복사해서 사용)
    - 불변 객체로 만드는 것이 안전

- 값 타입은 정말 값 타입이라 판단될 때만 사용
- 엔티티와 값 타입을 혼동해서 엔티티를 값 타입으로 만들면 안됨
- 식별자가 필요하고, 지속해서 값을 추전, 변경해야 한다면 그것은 값 타입이 아닌 엔티티

```java

public class Member {
  public static void main(String[] args) {
    Member member = new Member();

    member.setName("Member1");
    member.setHomeAddress(new Address("city1", "Street", "zip code"));

    member.getFavoriteFood().add("치킨");
    member.getFavoriteFood().add("족발");
    member.getFavoriteFood().add("피자");

    member.getAddressHistory().add(new Address("city2", "Street 1", "zip code 1"));
    member.getAddressHistory().add(new Address("city3", "Street 2", "zip code 2"));

    entityManager.persist(member);
    
    entityManager.flush();
    entityManager.clear();

    Member findMember = entityManager.find(Member.class, member.getId());

    List<Address> addressList = findMember.getAddressHistory();
    for (Address address: addressList){
      System.out.println("address.getCity() = " + address.getCity());
    }
  }
}
```

```java

@Entity
public class Member {
  @Id
  @Column(name = "MEMBER_ID")
  @GeneratedValue
  private Long id;

  @Column(name = "USERNAME")
  private String name;

  @Embedded
  private Period workPeriod;

  @Embedded
  private Address homeAddress;

  @ElementCollection
  @CollectionTable(name = "FAVORITE_FOOD", joinColumns = @JoinColumn(name = "MEMBER_ID"))
  @Column(name = "FOOD_NAME")
  private Set<String> favoriteFood = new HashSet<>();

  @ElementCollection
  @CollectionTable(name = "ADDRESS", joinColumns = @JoinColumn(name = "MEMBER_ID"))
  private List<Address> addressHistory = new ArrayList<>();
}

```

- 실사용 예제

```java

@Embeddable
public class Address {

    @Column(length = 50)
    private String city;
    @Column(length = 250)
    private String street;
    @Column(length = 250)
    private String zipcode;

    private void setCity(String city) {
        this.city = city;
    }

    private void setStreet(String street) {
        this.street = street;
    }

    private void setZipcode(String zipcode) {
        this.zipcode = zipcode;
    }

    public String getCity() {
        return city;
    }

    public String getStreet() {
        return street;
    }

    public String getZipcode() {
        return zipcode;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Address address = (Address) o;
        // Proxy 일 때도 정상동작하려면 getter 를 사용하도록 해야한다.
        // Proxy가 많이 사용되지 않을 경우에는 그냥해도 되지만 그냥 equals & hashcode는 무조건 짜라
        return Objects.equals(getCity(), address.getCity()) && Objects.equals(getStreet(), address.getStreet()) && Objects.equals(getZipcode(), address.getZipcode());
    }

    @Override
    public int hashCode() {
        return Objects.hash(getCity(), getStreet(), getZipcode());
    }
}

```

```java

@Entity
public class Member {

    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    private String name;

    @Embedded
    private Address address;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Address getAddress() {
        return address;
    }

    public void setAddress(Address address) {
        this.address = address;
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
        city varchar(50),
        street varchar(250),
        zipcode varchar(250),
        name varchar(255),
        primary key (MEMBER_ID)
    )
```