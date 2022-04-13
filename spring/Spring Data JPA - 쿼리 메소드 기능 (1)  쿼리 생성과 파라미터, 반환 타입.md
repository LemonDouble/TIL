# Spring Data JPA - 쿼리 메소드 기능 (1) : 쿼리 생성과 파라미터, 반환 타입

# 1. 메소드 이름으로 쿼리 생성

- 예를 들어, 상품명으로 조회 등의 Domain 특화된 기능은 어떻게 제공하면 좋을까?

### 1) Method 이름으로 쿼리 제공

- JPA로 직접 짠다면 이런 메소드를 직접 JPQL로 짜야 한다.

```java
public List<Member> findByUsernameAndAgeGreaterThen(String username, int age){
        return em.createQuery("select m from Member m where m.username = :username and m.age > :age")
                .setParameter("username", username)
                .setParameter("age", age)
                .getResultList();
    }
```

- 하지만 Spring Data JPA 사용하면 Method 이름만으로도 같은 기능을 제공한다!

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
    List<Member> findByUsernameAndAgeGreaterThan(String username, int age);
}
```

- 자세한 내용은 공식 문서 참고
    - 쿼리 메소드 필터 조건 : [Link](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods.query-creation)

- Spring Data JPA가 제공하는 쿼리 메소드 기능
    - 조회 : find...By, read...By, query...By, get...By + 필터 조건
        - 필터 조건 없을 시 전체 조회!
        - 설명을 위해 ...에는 아무것이나 들어갈 수 있다. (findHelloBy)
    - Count : count...By , 반환타입은 Long
    - Exist : exists...By, 반환타입은 Boolean
    - 삭제 : delete...By, remove...By 반환타입은 Long
    - DISTINCT(중복 제거): findDistinct, findMemberDistinctBy
    - LIMIT : findFirst3, findFirst, findTop, findTop3 ( 조건 : [Link](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories.limit-query-result) )
    
- 기억할 점!
    - Entity 필드 명 변경시 메소드명도 변경시켜 줘야 한다.
    - 만약 매칭 안 되는 경우, 어플리케이션 시작 시점에서 오류 발생
    - 어플리케이션 시작 시점에서 오류를 찾을 수 있는게 큰 장점이다!

# 2. JPA NamedQuery

- JPA는 Query의 이름을 부여하고 호출할 수 있는 기능을 제공
- 쿼리의 이름을 부여하고, 재사용 하겠다는 의미!

```java
@Entity
@NamedQuery(
	name="Member.findByUsername",
	query="select m from Member m where m.username = :username")
public class Member{
	...
}

// 이후 호출 가능
em.createNamedQuery("Member.findByUserName", Member.class)
	.setParameter("username", username)
	.getResultList();
```

- NamedQuery 또한 마찬가지로, 어플리케이션 실행 시점에 오류가 발생하는 장점이 있다.
- 하지만 NamedQuery 사용하는 경우보다, 차라리 @Query를 리포지토리에 직접 정의하는 경우가 더 많다.
- 이런게 있단거 정도만 알아두자..

# 3. @Query, 리포지토리 메소드에 쿼리 정의하기

- Repository 클래스에 다음과 같이 직접 쿼리 작성 가능!

```java
@Query("select m from Member m where m.username = :username and m.age = :age")
List<Member> findUser(@Param("username") String username, @Param("age") int age);
```

- 마찬가지로, 에러 있을 시 어플리케이션 실행 시점에 오류가 발생한다.
- 간단한건 메소드 이름으로, 복잡한건 @Query 쓰자!

# 4. @Query를 통해 값, DTO 조회하기

- 값 하나를 조회 (JPA의 @Embedded 타입도 이렇게 조회할 수 있다.)

```java
@Query("select m.username from Member m)
List<String> findUsernameList();
```

- DTO로 직접 조회
    - JPA의 new 명령어를 사용
    - 또한 Parameter가 맞는 생성자를 가진 DTO가 필요하다!

```java
@Query("select new study.datajpa.dto.MemberDto(m.id, m.username, t.name) " + 
"from member m join m.team t")
List<MemberDto> findMemberDto();
```

# 5. 파라미터 바인딩

- 위치 기반과 이름 기반

```sql
select m from Member m where m.username =?0 // 위치 기반
select m from Member m where m.username =:name // 이름 기반 
```

- 가능한 이름 기반을 사용하자!
    - 위치 기반은 쿼리가 변경되거나 할 때, 예상치 못한 작동 할 수 있다.

- 또한 Collection 타입으로 in절을 지원한다.

```sql
@Query("select m from Member m where m.username in :names")
List<Member> findByNames(@Param("names") List<String> names);
```

# 6. 반환 타입

- Spring Data JPA는 Collection, Optional, 일반 인스턴스 등 다양한 반환 타입을 지원 ( [Link](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repository-query-return-types) )
- 단순히 Repository에서 다음과 같이 작성하면 된다.

```java
List<Member> findListByUsername(String username); // Collections
Member findMemberByUsername(String username); // 단건, 없을시 null
Optional<Member> findOptionalByUsername(String username); // Optional
```

- 가능하면 단건 조회는 Optional 쓰자!
    - Optional이나 인스턴스 단건 조회시, 2개 이상이면 Excpetion 생긴다.

- Spring Data JPA는 JPA에서 Exception을 발생시키면, Spring Exception으로 변환해 주는 역할도 한다.
    - Spring이 추상화 해 줌으로써, JPA, Redis 등등 하위 Client와 비즈니스 로직의 의존성이 줄어듬

- @Async 어노테이션을 통해 비동기 처리도 가능하다! (알고만 있자)