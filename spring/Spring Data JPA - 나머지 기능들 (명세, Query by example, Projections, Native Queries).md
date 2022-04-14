# Spring Data JPA - 나머지 기능들 (명세, Query by example, Projections, Native Queries)

- 다른 대안들도 있지만, 이런거 있다 정도만 알아두자.

# 1. Specifications (명세)

- DDD의 Specification(명세) 라는 개념
    - Jpa Criteria 활용해서 사용 가능하게 지원
    
- 술어(Predicate)
    - True / False로 평가됨
    - AND, OR같은 연산자로 조합해 다양한 검색조건을 쉽게 생성
    - org.springframework.data.jpa.domain.Specification 클래스로 정의

```java
// Specification 잘 정의하면
Specification<Member> spec = MemberSpec.username("m1").and(MemberSpec.teamName("teamA"));

// 아래와 같이 사용 가능
memberRepository.findAll(spec)
```

- Criteria 좀 많이 복잡하다.... 써야 할 일 생기면 보자...

# 2. Query By Example ([Link](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#query-by-example))

- 다음과 같이, 객체를 사용하여 검색이 가능하다.

- 용어 정리
    - Probe : 필드에 데이터가 있는 실제 도메인 객체
    - ExampleMatcher : 특정 필드를 일치시키는 상세한 정보
    - Example : Probe + ExampleMatcher. 쿼리를 생성하는데 사용

```java
Member member = new Member("m1");
Team team = new Team("teamA"); //내부조인으로 teamA 가능
member.setTeam(team);
//ExampleMatcher 생성, age 프로퍼티는 무시
ExampleMatcher matcher = ExampleMatcher.matching()
.withIgnorePaths("age");
Example<Member> example = Example.of(member, matcher);
List<Member> result = memberRepository.findAll(example);
//then
assertThat(result.size()).isEqualTo(1);
```

- 치명적인 단점
    - 외부 조인 (Left Join) 불가
    - Inner Join은 가능하지만 복잡성이 높아지면 해결이 불가능한 문제가 있다.

# 3. Projections ([Link](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#projections))

- Projection : Query Select 절에 들어가는 Data 비스무리한 무언가..

- Entity 대신 DTO를 편하게 조회하고 싶을 때 사용
    - 회원 Entity에 Field가 40갠데, 내가 필요한 데이터가 이름 Field 하나 뿐이면?

### Close Projection (Proxy 사용해 작동된다.)

1. Interface를 정의

```java
public interface UsernameOnly {
    String getUsername();
}
```

1. Repository에서 해당 Interface를 반환하도록 설정

```java
//projection 사용
List<UsernameOnly> findProjectionsByUsername(@Param("username") String username);
```

1. 이후 다음과 같이 사용 가능하다.

```java
List<UsernameOnly> result = memberRepository.findProjectionsByUsername("m1");

for (UsernameOnly usernameOnly : result) {
    System.out.println("usernameOnly.getUsername() = " + usernameOnly.getUsername());
}
```

- 나가는 SQL 봤을 때, 다음과 같이 username만 받아온다.

```sql
select
        member0_.username as col_0_0_ 
    from
        member member0_ 
    where
        member0_.username=?
```

### Open Projection

- 다음과 같이, SpEL 을 사용할 수도 있다.
    - 단, 이 경우는 Entity를 전부 가져온 뒤 요구한 Field만 남기는 방법으로 작동한다.
    - 따라서, Select절 최적화는 기대할 수 없다!

```java
public interface UsernameOnly {
 @Value("#{target.username + ' ' + target.age + ' ' + target.team.name}")
 String getUsername();
}
```

### Class 기반 Projection

- Select절 최적화가 된다!

```java
public class UsernameOnlyDto {

 private final String username;

 public UsernameOnlyDto(String username) { // 매칭 될 때, 생성자의 Parameter 기준으로 매칭됨!
 this.username = username;
 }
 public String getUsername() {
 return username;
 }
}

// Repository에는 이렇게
List<UsernameOnlyDto> findClassProjectionsByUsername(@Param("username") String username);
```

### Generic 기반 Projection (동적 Projection)

- Repository 에서 다음과 같이 Generic 지정

```java
//Generic Type을 사용해서 Type도 Parameter로 전달하도록 변경
<T> List<T> findGenericProjectionsByUsername(@Param("username") String username, Class<T> type);
```

- 다음과 같이 사용할 수 있다!

```java
List<UsernameOnlyDto> resultDto = memberRepository.findGenericProjectionsByUsername("m1", UsernameOnlyDto.class);
List<UsernameOnly> resultInterface = memberRepository.findGenericProjectionsByUsername("m1", UsernameOnly.class);

for (UsernameOnly usernameOnly : resultInterface) {
    System.out.println("usernameOnly = " + usernameOnly);
}

for (UsernameOnlyDto usernameOnlyDto : resultDto) {
    System.out.println("usernameOnlyDto = " + usernameOnlyDto);
}
```

### 중첩 Projection

- Team 정보도 같이 가져오고 싶다면?

```java
// 이름은 상관없음
public interface NestedClosedProjections {
    String getUsername();
    TeamInfo getTeam();

    interface TeamInfo{
        String getName();
    }
}
```

- 다음과 같이 사용 (위의 Generic 사용)

```java
List<NestedClosedProjections> resultDto = memberRepository.findGenericProjectionsByUsername("m1", NestedClosedProjections.class);

for (NestedClosedProjections nestedClosedProjections : resultDto) {
    System.out.println("nestedClosedProjections.getUsername() = " + nestedClosedProjections.getUsername());
    System.out.println("nestedClosedProjections.getTeam() = " + nestedClosedProjections.getTeam());
}
```

- SQL 찾아보면?

```sql
select
        member0_.username as col_0_0_,
        team1_.team_id as col_1_0_,
        team1_.team_id as team_id1_2_,
        team1_.name as name2_2_ 
    from
        member member0_ 
    left outer join
        team team1_ 
            on member0_.team_id=team1_.team_id 
    where
        member0_.username=?
```

- Team의 경우는 Entity를 그대로 전부 가져온다.
    - 즉, Root( 이 경우는 Member) 는 Select절 최적화되는데, Join 대상은 최적화 안 된다!
    - Projection 대상이 Root 아닌 경우, Left Outer Join으로 처리한다.

- 결론 : Projection 대상이 Root Entity면 유용하다.
    - 하지만 Join해야 되는 경우, Select 최적화가 안 된다.
    - 간단한 경우에만 사용하자.

# 4. Native Query

- SQL 그대로 사용하는 것
    - 가능한 사용하지 않는 것 권장, 정말 어쩔 수 없을 때만 사용

- Repository에서 다음과 같이 사용 가능하다.
    
    ```sql
    //Native Query 사용
    @Query(value = "select * from member where username= ?", nativeQuery = true)
    Member findByNativeQuery(String username);
    ```
    
- Native Query
    - 페이징 지원
    - 반환 타입 : Object[], Tuple , Dto (Projections)
    - 제약
        - Sort 파라미터 동한 정렬이 정상 동작하지 않을 수 있다.
        - JPQL처럼 애플리케이션 로딩 시점에 문법 확인 불가 (!!)
        - 동적 쿼리 처리 불가

- 제한으로 인해, Native SQL을 DTO로 조회하려면 JdbcTemplate나 mybatis, QueryDSL 등 사용 권장

### Projections 활용

1. Projection Interface 선언

```java
public interface MemberProjection {
    Long getId();
    String getUsername();
    String getTeamName();
}
```

1. JPARepository에서 다음과 같이 선언

```java
//Native Query + Projections, count Query 꼭 짜줘야 한다!
@Query(value = "select m.member_id as id, m.username, t.name as teamName from member m left join team t",
        countQuery = "select count(*) from member",
        nativeQuery = true)
Page<MemberProjection> findByNativeProjection(Pageable pageable);
```

1. 다음과 같이 사용할 수 있다!
```java
Page<MemberProjection> result = memberRepository.findByNativeProjection(PageRequest.of(0, 10));
List<MemberProjection> content = result.getContent();

for (MemberProjection memberProjection : content) {
    System.out.println("memberProjection.getUsername() = " + memberProjection.getUsername());
    System.out.println("memberProjection.getTeamName() = " + memberProjection.getTeamName());
}
```