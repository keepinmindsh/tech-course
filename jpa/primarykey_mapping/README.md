## 기본키 매핑 

- **기본키 자동 생성 전략 4가지** 
  - IDENTITY
  - SEQUENCE
  - TABLE
  - AUTO

### 직접할당 

- @Id 만 사용 

### 자동 생성 

- @Id 와 @GeneratedValue를 같이 사용 

## IDENTITY

- 기본키 생성을 데이터 베이스에 위임 
  - 주로 MySQL, PostgreSQL, SQL Server, DB2 에서 사용 
- JPA는 보통 트랜잭션 커밋 시점에 INSERT SQL 실행 
  - AUTO_INCREMENT 는 데이터베이스에 INSERT SQL을 실행한 이후에 ID 값을 알 수 있음
- IDENTITY 전략은 em.persist() 시점에 즉시 INSERT_SQL 실행하고 DB 에서 식별자 조회 

- IDENTITY 전략 
  - 기본키 생성을 데이터베이스에 위임
    - 영속성 컨텍스트를 위해서 PK 값이 있어야 아는데, PK 값을 알 수 있는 시기가 Commit 이후임.
  - 주로 MySQL, PostgreSQL, SQL Server, DB2에서 사용 ( 예: MySQL의 AUTO_INCREMENT )
  - JPA 는 보통 트랜잭션 커밋 시점에 INSERT SQL 실행 
  - AUTO_INCREMENT 는 데이터베이스에 INSERT_SQL을 실행한 이후에 ID 값을 알 수 있음 
  - IDENTITY 전략은 em.persist() 시점에 즉시 INSERT_SQL 실행하고 DB에서 식별자 조회 
    - identity 전략은 persist() 시점에 insert 값을 처리하고 pk 값을 확인할 수 있다. 
    
```java

@Entity
public class MemberForSeq {
    
  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  private Long id;

}

```

## SEQUENCE 

- 데이터 베이스 시퀀스는 유일한 값을 순서대로 생성하는 특별한 데이터 베이스 오브젝트 ( 예 : 오라클 시퀀스 )
  - 오라클, PostgreSQL, DB2, H2 데이터베이스에서 사용 
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

## TABLE 

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

## @SequenceGenerator

- name : 식별자 생성기 이름 
- sequenceName : 데이터베이스에 등록되어 있는 시퀀스 이름 
- initialValue : DDL 생성 시에만 사용됨, 시퀀스 DDL을 생성할 때 처음 1 시작하는 수를 지정한다. 
- allocationSize : 시퀀스 한 번 호출에 증가하는 수(성능 최적화에 사용됨). 데이터베이스 시퀀스 값이 하나씩 증가하도록 설정되어 있으면 이 
  값을 반드시 1로 설정해야한다. ! 주의 - 기본값 : 50
- catalog, schema : 데이터베이스 catalog, schema 이름 
  
- SEQUENCE 전략
  - Commit 이전에 Sequences 에 의해서 미리 할당해둔 값 까지 성능 최적화를 위해서 가져올 수 있게 처리한다. 
  - 미리 값을 올려두는 방식이기 때문에 크게 상관이 없음 

```java
@Entity
@SequenceGenerator(name="member_seq_generator", sequenceName = "member_seq", allocationSize = 1)
public class MemberForSeq {

  @Id
  @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "member_seq_generator")
  private Long id;


}
```

## 권장하는 식별자 전략

- 기본키 제약 조건 : null 아님, 유일, 변하면 안된다. 
- 미래까지 이 조건을 만족하는 자연키를 찾기 어렵다. 대리키를 사용하자. 
- 예를 들어 주민등록번호도 기본 키로 적절하지 않다. 
- 권장방식 : Long 형 + 대체키 + 키 생성 전략 사용  
  
