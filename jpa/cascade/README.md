### 영속성 전이 : CASCADE

- 특정 엔티티를 연속 상태로 만들 때 연관된 엔티티도 함께 영속 상태로 만들고 싶을 때
    - 예: 부모 엔티티를 저장할 때 자식 엔티티도 함께 저장

- 예를 들어, 아래와 같은 코드를 작성한다고 할 때, entityManager를 3번이나 호출해야하는데, 실질적으로 따지고 보면 parent 시점에 이미 child가 추가되었는데 왜 각각 해줘야하는지에 대한 고민을 할 수 있음.

```java
public class JPAMain {
    public static void main(String[] args) {
        
        Child child1 = new Child();
        Child child2 = new Child();

        Parent parent = new Parent();
        parent.addChild(child1);
        parent.addChild(child2);

        entityManager.persist(parent);
        entityManager.persist(child1);
        entityManager.persist(child2);

        entityTransaction.commit();

    }
}

```

- 아래와 같이 자식 객체에 대해서 cascade를 정의하고, persist를 parent에 대해서만 실행하면 persist를 3번 호출할 필요는 없다.

```java

@Entity
public class Parent {

    @Id
    @GeneratedValue
    private Long id;

    private String name;

    @OneToMany(fetch = FetchType.LAZY, cascade = CascadeType.ALL)
    private List<Child> childList = new ArrayList<>();

    public void addChild(Child child){
        childList.add(child);
        child.setParent(this);
    }
}

```

```java

public class JPAMain {
    public static void main(String[] args) {
        Child child1 = new Child();
        Child child2 = new Child();

        Parent parent = new Parent();
        parent.addChild(child1);
        parent.addChild(child2);

        entityManager.persist(parent);
    }
}
```

- CASCADE의 종류 : ALL, PERSIST 정도를 실무에서 많이 활용하는 편임.
    - ALL : 모두 적용
    - PERSIST : 영속
    - REMOVE : 삭제
    - MERGE : 병합
    - REFRESH : REFRESH
    - DETACH : DETACH
- 주의할 점
    - 부모가 정말 하나의 부모가 자식들을 관리할 때는 의미가 있음
        - 게시판과 첨부파일의 데이터 경로
    - 파일을 여러곳에서 관리하는 엔티티의 경우에는 사용해서는 안됨.
        - CHILD가 여러 PARENT와 연관관계를 가질 경우에는 사용하지 말것, 소유자가 하나일 경우에만 사용할 것

### 고아 객체

- 고아 객체 제거 : 부모 엔티티와 연관관계가 끊어진 자식 엔티티를 자동으로 삭제
- orphanRemoval = true
- Parent parent1 = em.find(Parent.class, id);
  parent1.getChildren().remove(0);
  // 자식 엔티티를 컬렉션에서 제거
- 참조가 제거된 엔티티는 다른 곳에서 참조하지 않는 고아 객체로 보고 삭제하는 기능
- 참조하는 곳이 하나인 경우에서만 사용해야함.
- 특정 엔티티가 개인 소유할 때 사용
- @OneToOne, @OneToMany만 가능
- 참고 : 개념적으로 부모를 제거하는 자식은 고아가 된다. 따라서 고아 객체 제거 기능을 활성화하면 부모를 제거할때 자식도 함께 제거된다.
    - 이것은 CascadeType.REMOVE 처럼 동작한다.

```java

@Entity
public class Parent {

    @Id
    @GeneratedValue
    private Long id;

    private String name;

    @OneToMany(fetch = FetchType.LAZY, cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Child> childList = new ArrayList<>();

    public void addChild(Child child){
        childList.add(child);
        child.setParent(this);
    }
}

```

```shell
 select
        c1_0.Parent_id,
        c1_1.id,
        c1_1.name,
        p1_0.id,
        p1_0.name 
    from
        Parent_Child as c1_0 
    left outer join
        Child as c1_1 
            on c1_0.childList_id = c1_1.id 
    left outer join
        Parent as p1_0 
            on c1_1.parent_id = p1_0.id 
    where
        c1_0.Parent_id = ?
Hibernate: 
    /* delete collection bong.lines.jpashoping.cascade.Parent.childList */ delete 
        from
            Parent_Child 
        where
            Parent_id=?
Hibernate: 
    /* insert collection
        row bong.lines.jpashoping.cascade.Parent.childList */ insert 
        into
            Parent_Child
            (Parent_id, childList_id) 
        values
            (?, ?)
Hibernate: 
    /* delete bong.lines.jpashoping.cascade.Child */ delete 
        from
            Child 
        where
            id=?
```

- Parent가 삭제되었을 때, 자동으로 child 객체도 삭제됨 ( orphanRemoval = true)
    - cascade = CascadeType.ALL , CascadeType.REMOVE가 동일한 효과를 가짐

```java
public class JPAMain {
    public static void main(String[] args) {
        EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("hello");

        EntityManager entityManager = entityManagerFactory.createEntityManager();

        EntityTransaction entityTransaction = entityManager.getTransaction();

        entityTransaction.begin();

        try{
            Child child1 = new Child();
            Child child2 = new Child();

            Parent parent = new Parent();
            parent.addChild(child1);
            parent.addChild(child2);

            entityManager.persist(parent);
            entityManager.persist(child1);
            entityManager.persist(child2);

            entityManager.flush();
            entityManager.clear();

            Parent findParent = entityManager.find(Parent.class, parent.getId());

            entityManager.remove(findParent);

            entityTransaction.commit();
        }catch (Exception exception){
            exception.printStackTrace();
            entityTransaction.rollback();
        }finally {
            entityManager.close();
        }

        entityManagerFactory.close();
    }
}
```

```shell

Hibernate: 
    select
        c1_0.Parent_id,
        c1_1.id,
        c1_1.name,
        p1_0.id,
        p1_0.name 
    from
        Parent_Child as c1_0 
    left outer join
        Child as c1_1 
            on c1_0.childList_id = c1_1.id 
    left outer join
        Parent as p1_0 
            on c1_1.parent_id = p1_0.id 
    where
        c1_0.Parent_id = ?
Hibernate: 
    /* delete collection bong.lines.jpashoping.cascade.Parent.childList */ delete 
        from
            Parent_Child 
        where
            Parent_id=?
Hibernate: 
    /* delete bong.lines.jpashoping.cascade.Child */ delete 
        from
            Child 
        where
            id=?
Hibernate: 
    /* delete bong.lines.jpashoping.cascade.Child */ delete 
        from
            Child 
        where
            id=?
Hibernate: 
    /* delete bong.lines.jpashoping.cascade.Parent */ delete 
        from
            Parent 
        where
            id=?
```

- 영속성 전이와 고아객체를 동시에 사용할 경우
    - CascadeType.ALL + orphanRemovel = true
    - 스스로 생명주기를 관리하는 엔티티는 em.persist()로 영속화 em.remove()로 제거
    - 두 옵션을 모두 활성화하면 부모 엔티티를 통해서 자식의 생명주기도 관리할 수 있음
        - parent에 포함된 child에 대한 생명주기도 관리됨. child의 생명주기를 parent가 관리하는 구조가 된다.
    - 도메인 주도 설계(DDD)의 Aggregate Root 개념을 구현할 때 유용
    