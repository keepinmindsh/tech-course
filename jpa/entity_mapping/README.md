### 엔티티 매핑 

##### @Entity

- @Entity 가 붙은 클래스는 JPA가 관리한다.
    - 파라미터가 없는 public 또는 proctected 생성자 필요 
    - JPA Spec으로 규정됨. 
    - DB에 저장하고 싶은 필드에는 final을 사용할 수 없다. 

##### @Table 

- 속성 
  - name : 매핑할 테이블 이름 지정 
  - catalog : 데이터 베이스 catalog 매핑
  - schema : 데이터 베이스 schema 매핑
  - uniqueConstraints : DDL 생성 시에 유니크 제약 조건 생성 

##### Mapping Annotations

- @Column  - 컬럼 매핑
  - name
  - insertable : 등록 여부 
  - updatable  : 변경 여부 
  - nullable   : null 값의 허용 여부 설정 
  - unique     : @Table의 uniqueConstraint와 같이만 한 컬럼에 간단히 유니크 제약조건을 걸 때 사용한다. 
    - 잘 사용하지 않는 값 - unique 인덱스 생성시 , 유니크 제약조건을 이름이 반영이 어려워 @Table 에 같이 선언하여 사용한다. 
    - @Table(uniqueConstraint=~~~)
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
  - 최근에는 ~ LocalDate(년월)와 LocalDateTime ( 년월일) - 최신 버전의 경우 변수 선언서 데이터 타입을 LocalDate와 LocalDateTime를 적용하는 경우
    그에 맞게 호출됨. 아래의 test1과 test2를 참고할 것 
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

##### 데이터베이스 스키마 자동 생성 

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



