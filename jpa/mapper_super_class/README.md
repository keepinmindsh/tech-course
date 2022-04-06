### Mappped Super Class

- 상속관계 매핑과는 크게 관계가 없음.
- 공통 매핑 정보가 필요할 때 사용 ( id, name )
    - 객체의 입장에서 중복되는 멤버 변수가 계속 나올 경우, 귀찮음을 해결하기 위해서 부모 클래스에 정의해서 상속해서 사용하고 싶은 경우
- 상속 관계 매핑 아님
- 엔티티 아님, 테이블과 매핑 안됨
- 부모 클래스는 상속 받는 자식 클래스에 매핑 정보만 제공
- 조회, 검색 불가
- 직접 생성해서 사용할 일이 없으므로 추상 클래스 권장
- 테이블과 관계 없고, 단순히 엔티티가 공통으로 사용하는 매핑 정보를 모으는 역할
- 주로 등록일, 수정일, 등록자, 수정자 같은 전체 엔티티에서 공통으로 적용하는 정보를 모을 때 사용
- 참고 : @Entity 클래스는 엔티티나 @MappedSuperclass로 지정한 클래스만 상속 가능

```java
@MappedSuperclass
public class BaseEntity {

    private String createdBy;
    private LocalDateTime createdDate;
    private String modifiedBy;
    private LocalDateTime lastMofiedDate;
}

@Entity
public class Member extends BaseEntity {
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

```

```shell

 create table Member (
   MEMBER_ID bigint not null,
    createdBy varchar(255),
    createdDate timestamp(6),
    lastMofiedDate timestamp(6),
    modifiedBy varchar(255),
    city varchar(255),
    name varchar(255),
    orders varchar(255),
    street varchar(255),
    zipcode varchar(255),
    primary key (MEMBER_ID)
)

```