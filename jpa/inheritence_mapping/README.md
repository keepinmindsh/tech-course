### 상속관계 매핑

- 관계형 데이터 베이스는 상속 관계 없음
- 슈퍼타입, 서브타입 관계라는 모델링 기법이 객체 상속과 유사
- 상속 관계 매핑 : 객체의 상속과 구조와 DB의 서브 타입, 슈퍼타입 관계를 매핑하는 것
    - 슈퍼 타입 서브타입 논리 모델을 실제 물리 모델로 구현하는 방법
        - 각각 테이블로 변환 -> 조인 전략
        - 통합 테이블로 전환 -> 단일 테이블 전략
        - 서브 타입 테이블로 전환 -> 구현 클래스마다 테이블 전략

- 주요어노테이션
    - @Inheritance(strategy=InheritanceType.XXX)
        - JOINED : 조인 전략
        - SINGLE_TABLE : 단일 테이블 전략
        - TABLE_PER_CLASS : 구현 클래스마다 테이블 전략
    - @DiscriminatorColumn(name="DTYPE")
    - @DiscriminatorValue("XXX")

- Entity를 상속 받은 후 , 별도의 설정이 없을 경우 SINGLE_TABLE 전략이 적용됨. - 한 테이블에 모든 컬럼이 정의됨.

```java

@Entity
public class Item {

    @Id @GeneratedValue
    private Long id;

    private String name;
    private int price;
}

@Entity
public class Album extends Item{

  private String artist;

}

@Entity
public class Book extends Item {

  private String author;
  private String itbn;
}

@Entity
public class Movie extends Item {

  private String director;
  private String actor;
}

```

```shell

Hibernate: create sequence Item_SEQ start with 1 increment by 50
Hibernate: 
    
    create table Item (
       DTYPE varchar(31) not null,
        id bigint not null,
        name varchar(255),
        price integer not null,
        artist varchar(255),
        author varchar(255),
        itbn varchar(255),
        actor varchar(255),
        director varchar(255),
        primary key (id)
    )

```

- @Inheritance(strategy = InheritanceType.JOINED) 적용시
    - 우리가 정의한 설계 모델에 따라서 테이블이 생성됨.

```java

@Entity
@Inheritance(strategy = InheritanceType.JOINED)
public class Item {

    @Id @GeneratedValue
    private Long id;

    private String name;
    private int price;
}

```

```shell

create table Album (
       artist varchar(255),
        id bigint not null,
        primary key (id)
    )
Hibernate: 
    
    create table Book (
       author varchar(255),
        itbn varchar(255),
        id bigint not null,
        primary key (id)
    )
Hibernate: 
    
    create table Item (
       id bigint not null,
        name varchar(255),
        price integer not null,
        primary key (id)
    )
Hibernate: 
    
    create table Movie (
       actor varchar(255),
        director varchar(255),
        id bigint not null,
        primary key (id)
    )
Hibernate: 
    
    alter table if exists Album 
       add constraint FKcve1ph6vw9ihye8rbk26h5jm9 
       foreign key (id) 
       references Item
Hibernate: 
    
    alter table if exists Book 
       add constraint FKbwwc3a7ch631uyv1b5o9tvysi 
       foreign key (id) 
       references Item
Hibernate: 
    
    alter table if exists Movie 
       add constraint FK5sq6d5agrc34ithpdfs0umo9g 
       foreign key (id) 
       references Item

```

- JOINED 전략에 의한 Insert 시

```shell

Hibernate: 
    /* insert bong.lines.jpashoping.inheritance.Movie*/ 
    insert 
        into
    Item (name, price, id) 
    values
        (?, ?, ?)


    /* insert bong.lines.jpashoping.inheritance.Movie*/ 
    insert
        into
    Movie
        (actor, director, id)
    values
        (?, ?, ?)

```

- JOIN 전략에 의해서 구성된 테이블 조회시

```shell

select
        m1_0.id,
        m1_1.name,
        m1_1.price,
        m1_0.actor,
        m1_0.director 
  from
        Movie as m1_0 
  inner join
        Item as m1_1 
            on m1_0.id = m1_1.id 
  where
        m1_0.id = ?

```

- @DiscriminatorColumn
    - DTYPE이 생기고 기본이 Entity 명이 들어가게됨. 실질적으로 어떤 값을 통해서 들어오게 된 것인지에 대한 인지가 가능하다.
    - 운영상 DTYPE은 반드시 필요함
    - SINGLE_TABLE일 경우, 선언하지 않아도 자동 세팅됨.
    - properties
        - name : 컬럼명 변경도 가능함.

```java


@Entity
@Inheritance(strategy = InheritanceType.JOINED)
@DiscriminatorColumn
public abstract class Item {
    
    @Id @GeneratedValue
    private Long id;

    private String name;
    private int price;
}


```

```shell
  create table Item (
     DTYPE varchar(31) not null,
      id bigint not null,
      name varchar(255),
      price integer not null,
      primary key (id)
  )
```

- @DiscriminatorValue("A")
    - 부모클래스에 값이 들어갈 때, 자식 클래스에 해당 Annotations에 정의되어 있는 값으로 넣어줄 수 있음

```java

@Entity
@DiscriminatorValue("A")
public class Album extends Item{

    private String artist;

}


```

- SINGLE_TABLE 전략
    - 상속관계의 객체에 대해서 하나의 테이블에도 모두 구성할 경우
        - @Inheritance(strategy = InheritanceType.SINGLE_TABLE)
        - 성능이 잘 나올 수 있음 , 조인 불필요
        - 단일 테이블 전략에서는 DiscriminatorColumn 이 선언되지 않아도 DTYPE이 필수로 생성됨.


```shell
   create table Item (
       DTYPE varchar(31) not null,
        id bigint not null,
        name varchar(255),
        price integer not null,
        artist varchar(255),
        author varchar(255),
        itbn varchar(255),
        actor varchar(255),
        director varchar(255),
        primary key (id)
    )
```

- 구현 클래스마다 테이블 전략
    - 상속관계의 객체 구조를 제거함.
    - @Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
    - @DiscriminatorColumn이 적용되지 않음
        - 테이블 마다 생성되기 때문에 DTYPE을 이용한 구분이 불필요함.
    - 해당 전략은
        - Item 객체의 값을 가져오고 싶을 경우, 하위테이블을 모두 union all을 적용해서 전체 조회해야함.
        - 성능상의 이슈가 발생할 수 있음.

```java


@Entity
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
public abstract class Item {
    @Id @GeneratedValue
    private Long id;

    private String name;
    private int price;
}


```

```shell

Hibernate: 
    
    create table Album (
       id bigint not null,
        name varchar(255),
        price integer not null,
        artist varchar(255),
        primary key (id)
    )
Hibernate: 
    
    create table Book (
       id bigint not null,
        name varchar(255),
        price integer not null,
        author varchar(255),
        itbn varchar(255),
        primary key (id)
    )
Hibernate: 
    
    create table Movie (
       id bigint not null,
        name varchar(255),
        price integer not null,
        actor varchar(255),
        director varchar(255),
        primary key (id)
    )

```

- 장단점
    - JOIN 전략 :
        - 장점 : 정규화 되어 있음, 외래키 참조 무결성 제약조건 사용 가능, 저장 공간 효율화
            - 기본적인 전략으로 사용해야함.
        - 단점 : 조회시 조인을 많이 사용됨, 조회 쿼리가 복잡함, 데이터 저장시 INSERT SQL 2번 호출
    - SINGLE TABLE 전략 :
        - 장점 : 조인이 필요 없으므로 성능이 빠름, 조회쿼리가 단순함.
        - 단점 : 자식 엔티티가 매핑항 컬럼은 모두 null 허용 해야함.
        - 단점 : 단일 테이블에 모든 것을 저장하므로 테이블이 커질 수 있으므로 상황에 따라서 조회 성능이 오히려 느려질 수 있다.
    - 구현클래스마다 테이블 전략
        - 사용하지 말 것
        - 이전략은 데이터베이스 전문가와 설계자가 좋아하지 않은 전략 !
        - 장점 : 서브 타입을 명확하게 구분해서 처리할 때 효과적, not null 제약조건 사용 가능
        - 단점 : 여러 자식 테이블을 함께 조회할 때 성능이 느림, 자식 테이블을 통합해서 쿼리하기 어려움.