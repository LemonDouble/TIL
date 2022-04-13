# Spring Data JPA - 쿼리 메소드 기능 (3) : 벌크성 수정 쿼리, EntityGraph, Hint & Lock

# 1. 벌크성 수정 쿼리

- 한번에 여러 데이터를 수정, 실시간성이 중요하지 않은 경우

- 순수 JPA만을 사용하는 경우 다음과 같이 할 수 있다.

```java
// Return값 : Update 된 Column 개수
public int bulkAgePlus(int age){
    return em.createQuery("update Member m set m.age = m.age + 1 where m.age >= :age")
            .setParameter("age", age)
            .executeUpdate();
}
```

- 같은 역할을 Spring Data JPA는 다음과 같이 할 수 있다.
    - 이 때, Modifying Annotation을 넣어줘야 한다. (미사용시 Exception 발생)

```java
// Bulk Update
// Modifying Annotation이 있어야 Update를 실행할 수 있다!
@Modifying
@Query("update Member m set m.age = m.age + 1 where m.age >= :age")
int bulkAgePlus(@Param("age") int age);
```

### 단, bulk 연산은 영속성 컨텍스트와 관련해 문제 생길 수 있다!

- DB의 값은 Update 되었지만, 영속성 컨텍스트에는 과거의 값이 남아있을 수 있다. (같은 트랜잭션 내라면)
- 따라서, Bulk 연산 후 재조회해야 할 일이 생긴다면, 영속성 컨텍스트를 초기화하자!

- 혹은 다음과 같은 방법을 사용하고, 꼭 **Test** 작성하자.
    - @Modifying(clearAutomatically= true) 를 Repository Annotation 옵션에 추가 (기본값:false)
    - em.flush(), em.clear() 를 이용하여 영속성 컨텍스트를 초기화하자.
    - 영속성 컨텍스트에 값이 없을 때 (즉, Bulk 연산을 먼저 한 뒤) 로직을 실행하자.

### 꼭 Bulk 연산 아니더라도, Mybatis, QueryDSL 등 DB 직접 수정하는 로직 있다면, 영속성 컨텍스트를 꼭 초기화하자.

# 2. @EntityGraph

- 연관된 Entity들을 SQL 한번에 조회 (Fetch Join을 편하게 사용할 수 있게 한다!)

- 아래와 같이, fetch Join 사용하거나 EntityGraph를 이용해 좀 더 쉽게 사용할 수 있다.

```java
//Fetch Join 사용해서 N+1 문제 해결!
@Query("select m from Member m left join fetch m.team")
List<Member> findMemberFetchJoin();

// JPQL 대신 EntityGraph를 사용해 해결할 수도 있다.
@Override
@EntityGraph(attributePaths = {"team"})
List<Member> findAll();

// JPQL과 EntityGraph를 같이 사용할 수도 있다. (team을 Fetch Join한것과 같이 동작)
@EntityGraph(attributePaths = {"team"})
@Query("select m from Member m")
List<Member> findMemberEntityGraph();

// 메소드 이름 생성 전략과 같이, EntityGraph도 사용할 수 있다.
@EntityGraph(attributePaths = {"team"})
  List<Member> findEntityGraphByUsername(@Param("username") String username);
```

# 3. JPA Hint & Lock

### JPA Hint

- JPA 쿼리 힌트 ( JPA 구현체 (Hibernate) 에게 제공하는 힌트)

- Repository에 Hint를 줄 수 있다.

```java
// Hibernate에게 SQL 힌트를 준다. readonly이므로 최적화의 여지가 있다.
@QueryHints(value = @QueryHint(name="org.hibernate.readOnly", value = "true"))
Member findReadOnlyByUsername(String username);
```

- readOnly 시에, 변경 감지는 작동하지 않는다!

```java
Member findMember = memberRepository.findReadOnlyByUsername("member1");
findMember.setUsername("member2");

// 원래 Flush 하면 변경 감지 되어야 하지만, readOnly Hint 줬으므로 변경 감지되지 않는다.
em.flush();
```

- 하지만 쓸 일이 그렇게 많진 않음
    - 만약 트래픽이 정말 커서 조회가 문제가 된다면, Redis 등의 Cache를 먼저 고려했을 것
    - 실제로 Bottleneck인 부분은 복잡한 조회 Query일 가능성이 높다.
    - 성능 테스트를 해 보고, 가장 Bottleneck이 큰 부분을 개선하자.

### JPA Lock

- 다음과 같이 Annotation 기반으로 락을 걸 수 있다.

```java
//select for update 나간다!
// 비관적 Lock
@Lock(LockModeType.PESSIMISTIC_WRITE)
List<Member> findLockByUsername(String username);
```

- 가능하면 실시간 서비스에서는 Lock을 최소한으로 사용하자
    - 정말 꼭 필요하면 낙관적 락을 차라리 사용하자. (처리량 이슈)
    - 가능하면 Lock을 안 거는게 더 좋다!