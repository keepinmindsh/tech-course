# **JPA 이해하기** 

## **JPA : Java Persistence API** 

JPA는 Java Persistence API의 약자로, 자바 어플리케이션에서 관계형 데이터베이스를 사용하는 방식을 정의한 인터페이스이다. 여기서 중요하게 여겨야 할 부분은, JPA는 말 그대로 인터페이스라는 점이다. JPA는 특정 기능을 하는 라이브러리가 아니다. 마치 일반적인 백엔드 API가 클라이언트가 어떻게 서버를 사용해야 하는지를 정의한 것처럼, JPA 역시 자바 어플리케이션에서 관계형 데이터베이스를 어떻게 사용해야 하는지를 정의하는 한 방법일 뿐이다.

- JPA는 인터페이스의 모음   

- JPA는 애플리케이션과 JDBC 사이에서 동작

- JPA 구현체 
    - Hibernate, EclipseLink, DataNucleus, OpenJPA, ObjectDB, TopLink Essential

## **H2Database 설정** 

https://www.h2database.com/html/main.html 를 이용한 H2 Database 구성 

## **JPA Main**

```java

package bong.lines.sample;

import javax.persistence.*;

public class HelloWorldJPA {
    public static void main(String[] args) {

        EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("hello");

        EntityManager entityManager = entityManagerFactory.createEntityManager();

        EntityTransaction entityTransaction = entityManager.getTransaction();

        entityTransaction.begin();

        try{

            Member member = new Member();
            member.setId(2L);
            member.setName("Hello");
            entityManager.persist(member);

        }catch (Exception exception){
            entityTransaction.rollback();
        }finally {
            entityManager.close();
        }

        entityManagerFactory.close();
    }
}


```

## **Persistence.xml**

- persistence.xml 설정 
    - jpa의 경우, 기본적인 설정 파일을 필요로 함. 순수 자바 프로젝트에서 사용할 경우 META-INF 하위에 persistence.xml 를 위치시키는 것이 중요함.
    - 항목별 설명
        - hibernate.dialect : 데이터베이스 방언, 다양하게 존재하는 DB의 DDL,DML 등의 특성, 고유한 기능에 맞춰 사용할 수 있게 해주는 설정
    - javax. ~ : 표준에서 제정한 것 
    - hibernate. ~ : 하이버네이트 전용 옵션
    - hibernate.show_sql : query를 출력함
    - hibernate.format_sql : sql을 예쁘게
    - hibernate.use_sql_comments : 쿼리 실행사유를 표시

```xml

<?xml version="1.0" encoding="UTF-8"?>
<persistence version="2.2"
             xmlns="http://xmlns.jcp.org/xml/ns/persistence" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd">
    <persistence-unit name="hello">
        <class>bong.lines.sample.Member</class>
        <class>bong.lines.sample.MemberForSeq</class>
        <properties>
            <!-- 필수 속성 -->
            <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
            <property name="javax.persistence.jdbc.user" value="sa"/>
            <property name="javax.persistence.jdbc.password" value=""/>
            <property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/test"/>
            <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>

            <!-- 옵션 -->
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.format_sql" value="true"/>
            <property name="hibernate.use_sql_comments" value="true"/>
            <property name="hibernate.hbm2ddl.auto" value="update" />
        </properties>
    </persistence-unit>
</persistence>

```

## **Hibernate 기본 설정**

- JPA Java Project 구성시 Maven 모듈 ( Gradle Or Maven )
    - https://mvnrepository.com/artifact/org.hibernate/hibernate-entitymanager
    - https://mvnrepository.com/artifact/com.h2database/h2

```gradle
plugins {
    id 'java'
}

group 'org.example'
version '1.0-SNAPSHOT'

repositories {
    mavenCentral()
}

dependencies {

    // https://mvnrepository.com/artifact/com.h2database/h2
    implementation 'com.h2database:h2:2.0.204'
    // https://mvnrepository.com/artifact/org.hibernate/hibernate-entitymanager
    implementation 'org.hibernate:hibernate-entitymanager:6.0.0.Alpha7'

    testImplementation 'org.junit.jupiter:junit-jupiter-api:5.7.0'
    testRuntimeOnly 'org.junit.jupiter:junit-jupiter-engine:5.7.0'
}

test {
    useJUnitPlatform()
}
```

## **Database 스키마 자동 설정**

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



```shell

hibernate.hbm2ddl.auto=create 일 때, @Column(nullable = false, length = 500, unique = false) 는 아래와 같이 동작함.

Hibernate: 
    
    drop table if exists MBR cascade 
Hibernate: 
    
    create table MBR (
       id bigint not null,
        name varchar(500) not null,
        primary key (id)
    )

```

## **주의사항**

- 운영 장비에는 절대 create, create-drp, update 사용하면안된다.
- 개발 초기 단계는 create 또는 update
- 테스트 서버는 update 또는 validate
- 스테이징과 운영 서버는 validate or none