## **JPA 맛보기**

JPA의 구현체인 hibernate를 활용하여 JPA를 시작한다. 본격적인 JPA 시작 이전에 샘플 프로젝트를 이용해서 구성한다.  

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

```java

package bong.lines.jpashoping.step1;

import javax.persistence.*;

@Entity
public class Member {

    @Id
    @GeneratedValue
    @Column(name="MEMBER_ID")
    private Long id;

    @Column(name="USERNAME")
    private String username;

    @Column(name="TEAM_ID")
    private Long teamId;

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

    public Long getTeamId() {
        return teamId;
    }

    public void setTeamId(Long teamId) {
        this.teamId = teamId;
    }
}

```

```xml

<?xml version="1.0" encoding="UTF-8"?>
<persistence version="2.2"
             xmlns="http://xmlns.jcp.org/xml/ns/persistence" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd">
    <persistence-unit name="hello">

        <class>bong.lines.jpashoping.step1.Member</class>
      
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
            <property name="hibernate.hbm2ddl.auto" value="create" />
        </properties>
    </persistence-unit>
</persistence>

```

## References 

- JPA Java Project 구성시 Maven 모듈 ( Gradle Or Maven )
  - https://mvnrepository.com/artifact/org.hibernate/hibernate-entitymanager
  - https://mvnrepository.com/artifact/com.h2database/h2

- [hibernate properties, configuration](https://docs.jboss.org/hibernate/core/3.6/reference/en-US/html/session-configuration.html)