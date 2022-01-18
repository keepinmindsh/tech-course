# JPA 이해하기 

- JPA : Java Persistence API 

JPA는 인터페이스의 모음   

JPA는 애플리케이션과 JDBC 사이에서 동작

- H2Database 설정 

https://www.h2database.com/html/main.html 를 이용한 H2 Database 구성 

- JPA Main

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

- persistence.xml
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

- hibernate 기본 설정 
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

 