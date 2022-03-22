## **JPA : Java Persistence API**

JPA는 Java Persistence API의 약자로, 자바 어플리케이션에서 관계형 데이터베이스를 사용하는 방식을 정의한 인터페이스이다. 여기서 중요하게 여겨야 
할 부분은, JPA는 말 그대로 인터페이스라는 점이다. JPA는 특정 기능을 하는 라이브러리가 아니다. 마치 일반적인 백엔드 API가 클라이언트가 어떻게 
서버를 사용해야 하는지를 정의한 것처럼, JPA 역시 자바 어플리케이션에서 관계형 데이터베이스를 어떻게 사용해야 하는지를 정의하는 한 방법일 뿐이다.

- JPA는 인터페이스의 모음

- JPA는 애플리케이션과 JDBC 사이에서 동작

- JPA 구현체
    - Hibernate, EclipseLink, DataNucleus, OpenJPA, ObjectDB, TopLink Essential
    
## **JPA 기본 구성**

![JPA 기본 계층도](https://github.com/keepinmindsh/tech-course/blob/83e50da1d3cd9900e9ac1dd51d06fde64a4f7489/assets/jpastudy_0002.png)

## **JPA 를 왜 사용해야 하는가?**

- SQL 중심적인 개발애서 객체 중심으로 개발
  - 마치 자바 Collection 에 값을 넣었다가 제거하는 것과 같이 SQL를 작성할 수 있다. 

- JPA는 생산성, 유지보수, 패러다임의 불일치 해결, 성능, 데이터 접근 추상화와 벤더 독립성, 표준을 지키지 위해서 사용한다.

## **Object Relation Mapping 은 객체와 RDB 사이의 두 기둥 위에 있는 기술**

- 객체지향과 RDB의 적절한 균형을 잘 맞춰야 한다.
  - ORM은 RDB의 개념을 객체를 기반으로 하여 객체 기반의 개발을 용이하게 하는 것 
  - Java를 예를 들 경우 결국, JDBC API를 활용하여 ORM을 구성한 Hibernate이기 때문이다. 
  
- RDB에 대한 이해가 더욱더 중요하다. 
  - JPA를 활용한 객체지향 설계 방식을 활용하기에 앞서서 RDB에 대한 이해가 더 중요하다.     
    그 이유는 결국 JPA도 JDBC를 바탕으로 하여 DB를 객체화하는 것이기 때문에 DB 자체에서 제공하는 Index, Optimizer, ACID 특성, Transaction, 제약조건, 기본문법들에 대해서 이해해야 한다.  
