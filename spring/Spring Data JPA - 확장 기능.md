# Spring Data JPA - 확장 기능

# 1. 사용자 정의 리포지토리 구현

- 기본적으로, Spring Data JPA는 인터페이스만 정의, 구현체는 자동 생성됨
- 하지만 다양한 이유로 인터페이스 메소드를 직접 구현하고 싶다면?
    - JPA를 직접 사용하거나,
    - JDBC를 직접 사용
    - QueryDSL이나 MyBatis를 사용해야 하거나 등등..

- 기능 추가를 위한 방법
1. 새로운 Interface를 선언

```java
public interface MemberRepositoryCustom {
    List<Member> findMemberCustom();
}
```

1. 해당 Interface를 구현하는 Class 선언.

```java
@RequiredArgsConstructor
public class MemberRepositoryCustomImpl implements MemberRepositoryCustom{

    private final EntityManager em;

    @Override
    public List<Member> findMemberCustom() {
        return em.createQuery("select m from Member m")
                .getResultList();
    }
}
```

1. JPARepository가 1) 에서 만든 Interface를 구현하도록 추가

```java
public interface MemberRepository extends JpaRepository<Member, Long> , MemberRepositoryCustom{
```

- 단, 2)의 구현 클래스의 이름이 중요하다! 관례상의 이름을 맞춰줘야 작동한다.
    1. JPA Repository 이름 + Impl (예시 : MemberRepositoryImpl)
    2. 커스텀 Interface 이름 + Impl (예시 : MemberRepositoryCustomImpl)

- 만약, Impl 대신 다른 이름을 사용하고 싶다면 (비권장)
    - Java Config를 변경
    
    ```java
    @EnableJpaRepositories(basePackages = "study.datajpa.repository",
     repositoryImplementationPostfix = "Impl")
    ```
    

- 항상 사용자 정의 리포지토리를 사용해야 되는 것은 아님
    - 그냥 새 리포지토리 클래스를 만들고, 직접 구현해도 됨.
    - 만약 View에 맞춘 복잡한 Query 있는 Repository가 있다면?
        - 사용자 정의 리포지토리 사용해도 됨.
        - 하지만 View 전용 리포지토리와, 비즈니스 로직 관련 리포지토리 분리할 수도 있음
        - 분리하면, 복잡성이 낮아질 수도 있다!
    - 이런 경우, Spring Data JPA와는 관계 없이 별도로 동작함.
    - 유지보수 / 관리 등 여러가지 측면을 생각하고 적절한 것을 픽하자

# 2. Auditing

- Entity 생성/변경시, 변경한 사람과 시간을 추적하고 싶으면?
    - 등록일
    - 수정일
    - 등록자
    - 수정자
- 실제로 운영 시 굉장히 중요한 데이터!

### JPA Event를 사용하여 처리

1. @MappedSuperclass를 이용하여 공통 관심사를 분리한다.
    - @PrePersist, PostPersist, PreUpdate, PostUpdate 등 있다.

```java
// 실제 상속이 아니라, 이 Class의 속성을 다른 테이블에서 쓸 수 있게 해줌.
@Getter @Setter
@MappedSuperclass
public class JpaBaseEntity {

    // 생성 시간은 변경 불가능하도록 updateable 을 false로
    @Column(updatable = false)
    private LocalDateTime createdDate;
    private LocalDateTime updatedDate;

    // 저장 전에 실행된다.
    @PrePersist
    public void prePersist(){
        LocalDateTime now = LocalDateTime.now();
        this.createdDate = now;
        this.updatedDate = now;
    }

    @PreUpdate
    public void preUpdate(){
        this.updatedDate = LocalDateTime.now();
    }
}
```

1. 해당 관심사가 있는 Entity는 위 Class를 상속

```java
public class Member extends JpaBaseEntity{
```

1. 다음과 같이 사용된다.

```java
@Test
public void JpaEventBaseEntity() throws InterruptedException {
    //given
    Member member = new Member("member1");
    memberRepository.save(member); //@PrePersist 발생!

    Thread.sleep(100);
    member.setUsername("member2");
    em.flush(); //@PreUpdate 발생!
    em.clear();
    //when

    Member findMember = memberRepository.findById(member.getId()).get();
    //then

    System.out.println("findMember.getCreatedDate() = " + findMember.getCreatedDate());
    System.out.println("findMember.getUpdatedDate() = " + findMember.getUpdatedDate());
}
```

### Spring Data JPA로 문제를 해결하기

1. Annotation 기반으로 공통 관심사를 분리한다.

```java
@EntityListeners(AuditingEntityListener.class)
@Getter
@MappedSuperclass
public class BaseEntity {

    @CreatedDate
    @Column(updatable = false)
    private LocalDateTime createdDate;

    @LastModifiedDate
    private LocalDateTime lastModifiedDate;

    @CreatedBy
    @Column(updatable = false)
    private String createdBy;

    @LastModifiedBy
    private String lastModifiedBy;
}
```

1. 어플리케이션에 @EnableJpaAuditing 추가 (추가하지 않을 시 작동 X)
    - AuditAware을 반환하는 Bean을 만든다!

```java
//SpringApplication.class (최상위 클래스)

@EnableJpaAuditing
@SpringBootApplication
public class DataJpaApplication {

	public static void main(String[] args) {
		SpringApplication.run(DataJpaApplication.class, args);
	}

	@Bean
	public AuditorAware<String> auditorProvider(){
		return () -> Optional.of(UUID.randomUUID().toString());
	}
}
```

- 실제로는, Spring Security 쓰면 SecurityContext 등에서 ID 꺼내거나 하는 방식으로 연결한다.
    - 혹은 Session값을 넣거나..
1. 관심사가 있는 Entity에서 상속받도록 변경

```java
public class Member extends BaseEntity{
```

- 하지만 실제로는 등록일/수정일은 모든 테이블에서 필요
    - 하지만 등록자/수정자는 의미 없는 테이블도 많다.
    - 등록일/수정일 클래스 하나 만들고 (A)
    - A를 상속받아서 등록자/수정자를 추가한 자식 Class를 만들면 깔끔하게 처리할 수 있다.

# 3. Web 확장 - 도메인 클래스 컨버터

```java
@GetMapping("/members/{id}")
public String findMember(@PathVariable("id") Long id){
    Member member = memberRepository.findById(id).get();
    return member.getUsername();
}
// 도메인 클래스 컨버터로 단순화 가능하다.
@GetMapping("/members2/{id}")
public String findMember2(@PathVariable("id") Member member){
    return member.getUsername();
}
```

- 권장하진 않는다. 이런 거 있다 정도만..

- 단, 조회용으로만 사용할 것!!
    - Transaction이 없는 범위에서 Entity 조회했으므로, Entity 변경해도 DB에 반영되지 않는다.

# 4. Web 확장 - 페이징과 정렬

- 다음과 같이 Controller 추가

```java
@GetMapping("/members")
public Page<Member> list(Pageable pageable){
    return memberRepository.findAll(pageable);
}
```

- [http://localhost:8080/members?page=0](http://localhost:8080/members?page=0) 시 : 20개씩 꺼내서 보여준다. (기본값이 20개)
- [http://localhost:8080/members?page=0&size=3](http://localhost:8080/members?page=0&size=3) 시 : 3개씩 꺼내서 보여준다!
- [http://localhost:8080/members?page=0&size=3&sort=id,desc&sort=username,desc](http://localhost:8080/members?page=0&size=3&sort=id,desc&sort=username,desc)
    - 이런 식으로 Sorting 조건을 정리할 수도 있다

- 요청 파라미터 정리
    - page : 현재 페이지, (0부터 시작)
    - size : 한 페이지당 데이터 수
    - sort : 정렬 조건, 기본값은 asc

- 페이징 정보가 둘 이상일 때
    - @Qualifier를 이용해, 접두사로 구분 가능 (”접두사명_xxx”)
    
    ```java
    // Request : /members?member_page=0&order_page=1
    public String list(
    	@Qualifier("member") PageablememberPagealble,
    	@Qualifier("order") PageablememberPagealble)
    ```
    

- 페이징 정보를 DTO로 변환
    - 엔티티는 절대 노출시키지 말자..
    - 적절한 생성자 만들고, map 통해 DTO로 변환시키자.

```java
// DTO 생성자
public MemberDto(Member member){
    this.id = member.getId();
    this.username = member.getUsername();
}
// map으로 DTO 변환
return memberRepository.findAll(pageable).map(MemberDto::new);
```

- Page를 1부터 시작하기
    - 기본은 0부터, 1부터 시작하도록 변경하려면?
        1. Pageable, Page를 직접 클래스를 만들어 처리.
        2. spring.data.web.pageable.one-indexed-parameters를 true로 설정 (application.yml)
    
    - 하지만 2) 방법의 경우는 한계가 있다!
        - URI의 page는 -1 해서 1일때 0 page, 2일때 1 page 보여줘서 1부터 시작하는것처럼 보이지만..
        - 추가 page 관련 데이터는 0 base Index로 시작한다.
        
        ```java
        {
         "content": [
         ...
         ],
         "pageable": {
         "offset": 0,
         "pageSize": 10,
         "pageNumber": 0 //0 인덱스
         },
         "number": 0, //0 인덱스
         "empty": false
        }
        ```
        
        - 결론 : 그냥 0 base index 그대로 쓰는 게 제일 좋다..