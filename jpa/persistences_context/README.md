## **Persistence Context**

- 객체와 관계형 데이터베이스 매핑하기
- 영속성 컨텍스트
    - JPA를 이해하는데 가장 중요한 용어
    - "엔티티를 영구 저장하는 환경"이라는 뜻
    - 논리적인 개념
    - 눈에 보이지 않음
    - Entity Manager를 통해 영속성 컨텍스트에 접근
    - 1차 캐시
        - Entity 조회 , 1차 캐시
        - 객체를 조회할 때, 영속성 컨텍스트에서 1차 캐시를 질의하여 데이터가 존재한다면 캐시에 있는 값을 그대로 조회함
        - 하나의 Transaction 안에서만 동작하기 때문에 효율성의 이점이 크지 않음. 다만 비즈니스 로직이 복잡할 경우는 도움이 될 수 있음
        - 객체에 대해서 저장하는 persist 시점에도 1차 캐시로 등록이 가능하다.
    - 동일성 보장
        - 1차 캐시로 반복 가능한 읽기(REPEATABLE READ) 등급의 트랜잭션 격리 수준을 데이터 베이스가 아닌 애플리케이션 차원에서 제공
    - 트랜잭션을 지원하는 쓰기 지연
        - persist 시점에 데이터를 DB Insert 처리하지 않고 Commit 시점에 처리된다.
        - persist를 다수 호출 후에 commit 명령어에 의해서 실행되므로, jdbc batch insert 와 동일하게 동작한다.
            - bufferning을 모아서 적용 가능
    - 변경 감지 ( Dirty Checking )
        - 영속 컨텍스트에 의해서 Transaction Commit (flush) 시점에 entity와 snapshot를 비교하는데,
          entity와 snapshot을 비교하여 변경된 값에 대해서 update query를 생성하여 commit 시점에 database에 반영된다.
    - 지연 로딩 
