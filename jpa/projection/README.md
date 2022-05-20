# 프로젝션
- SELECT 절에 조회할 대상을 지정하는 것
- 프로젝션 대상 : 엔티티, 임베디드 타입, 스칼라 타입 ( 숫자, 문자 등 기본 데이터 타입 )
- SELECT m FROM Member m -> 엔티티 프로젝션
    - 조회된 모든 데이터에 대해서 영속성 계층에서 관리가 가능함.
- SELECT m.team FROM Member m -> 엔티티 프로젝션
    - 쿼리와 비슷하게 JPQL을 작성하는 것이 중요함.
        - Member에서 Team을 가져와야할 경우!
- SELECT m.address FROM Member m -> 임베디드 타입 프로젝션
    - Embed 타입은 Entity 내에서 정의되어서만 사용할 수 있음.
- SELECT m.username, m.age FROM Member m -> 스칼라 타입 프로젝션
    - 내가 원하는 데이터를 자유롭게 가져오는 것


# 프로젝션 - 여러값 조회

- SELECT m.username, m.age FROM Member m
- Query 타입으로 조회
- Object[] 타입으로 조회
- new 명령어로 조회
    - 단순 값을 DTO로 바로 조회
      SELECT new jpabook.jpql.UserDTO(m.username, m.age) FROM Member m
    - 패키지 명을 포함한 전체 클래스 명 입력
    - 순서와 타입이 일치하는 생성자 필요

```java

class Sample {
  public static void main(String[] args) {
    List<Member> result = entityManager.createQuery("select m.userName, m.age from Member m").getResultList();

    TypedQuery<Member> memberTypedQuery = entityManager.createQuery("SELECT m from Member  m", Member.class);
    List<Member> memberList = memberTypedQuery.getResultList();
    Member findMember = memberList.get(0);
    System.out.println("findMember.getUserName() = " + findMember.getUserName());
    System.out.println("findMember.getAge() = " + findMember.getAge());

    Query query = entityManager.createQuery("SELECT m.userName, m.age from Member m");

    List resultList = query.getResultList();
    Object o = resultList.get(0);
    Object[] resultObj = (Object[])o;
    System.out.println("resultObj[0] = " + resultObj[0]);
    System.out.println("resultObj[0] = " + resultObj[1]);

    List<Object[]> resultListWithCasting = query.getResultList();
    Object[] resultWithCasting = resultListWithCasting.get(0);
    System.out.println("resultObj[0] = " + resultWithCasting[0]);
    System.out.println("resultObj[0] = " + resultWithCasting[1]);

    List<MemberDTO> resultList1 = entityManager.createQuery("SELECT new bong.lines.jpql.sample.MemberDTO(m.userName, m.age) from Member m", MemberDTO.class).getResultList();

    MemberDTO selectedMemberDTO = resultList1.get(0);
    System.out.println("selectedMemberDTO.getUserName() = " + selectedMemberDTO.getUserName());
    System.out.println("selectedMemberDTO.getAge() = " + selectedMemberDTO.getAge());
  }
}
