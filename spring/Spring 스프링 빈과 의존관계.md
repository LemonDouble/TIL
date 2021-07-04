# Spring : 스프링 빈과 의존관계

분류: Spring
작성일시: 2021년 7월 4일 오후 4:53

## 의존성?

- MemberController가 MemberService 클래스를 통해 회원가입하고, 데이터를 조회하는 식으로 구현되어 있는 경우, MemberController는 MemberService 클래스에 의존관계가 있다고 한다.

## Spring bean

- Controller annotation이 있는 경우, Spring이 실행될 때 객체를 생성하여 Spring이 관리함.

- Spring 사용시, (controller/MemberController)

```java
private final MemberService memberService = new MemberService();
```

와 같이 new 해서 사용하지 않고

```java
private final MemberService memberService;

@Autowired
public MemberController(MemberService memberService) {
    this.memberService = memberService;
}
```

과 같이 Spring에 등록해서 사용한다.

이 때, MemberService가 Spring bean에 등록되지 않아 에러가 나는데,

(service/MemberService)

```java
@Service // << 추가!
public class MemberService {

    private final MemberRepository memberRepository;

    //Dependency injection!!
    public MemberService(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
}
...
...
...
```

위와 같이 @Service 어노테이션을 추가해 Spring bean에 등록을 해 준다.

마찬가지로, Repository에도

(repository/MemoryMemberRepository)

```java
@Repository
public class MemoryMemberRepository implements MemberRepository{

    private static Map<Long, Member> store = new HashMap<Long, Member>();
    //실무에서는 Concurrent 문제 때문에 ConcurrentHashmap 사용해야 함

    private static long sequence = 0L;
    //이것도 마찬가지로 실무에서는 동시성 고려해야함

...
...
...
```

과 같이 @Repository 어노테이션을 통해 Repository 등록을 해 준다.

- Tip : Spring은 Container에 Bean을 등록할 때, 기본적으로 싱글톤으로 등록한다. 따라서, 같은 Bean이면 같은 Instance이다. 설정으로 Singleton 아니게 할 수 있지만, 특별한 경우를 제외하곤 Singleton만 사용한다.

### 이 때 Controller의 @Autowired annotation → Spring이 Spring Bean에 등록되어 있는 객체를 삽입해 준다 : 이것을 의존성 주입이라고 함 (Dependency Injection)

## Spring bean의 등록 방법

1. Component Scan과 자동 의존관계 설정
    - 위에 사용한 방식, Annotation (@Component) 찾아 등록한다.
    - Controller, Service, Repository도 Component를 포함하고 있다.
    - @Autowired : 연관관계를 설정
2. 자바 코드로 직접 Bean 등록하기
    - Java/project명/SpringConfig (class) 생성한다. Spring이 실행할 때 해당 SpringConfig 파일을 읽고, MemberService, MemoryMemberRepository를 실행한다.

    ```java
    package rabbitprotocol.fileuploadrabbitprotocolbackend;

    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import rabbitprotocol.fileuploadrabbitprotocolbackend.repository.MemberRepository;
    import rabbitprotocol.fileuploadrabbitprotocolbackend.repository.MemoryMemberRepository;
    import rabbitprotocol.fileuploadrabbitprotocolbackend.service.MemberService;

    @Configuration
    public class SpringConfig {

        @Bean
        public MemberService memberService(){
            return new MemberService(memberRepository());
        }

        @Bean
        public MemberRepository memberRepository(){
            return new MemoryMemberRepository();
        }
    }
    ```

## 왜 자바 코드로 직접 Bean 등록을 사용하는가?

- 실무에서 정형화된 컨트롤러, 서비스, 리포지토리 같은 코드는 컴포넌트 스캔을 사용
- ** 하지만 정형화되지 않았거나, 상황에 따라 구현 클래스를 변경해야 하면 직접 등록을 한다. **

    (예시 : Database 바뀌는 경우 등)  

## DI (dependency injection) 의 종류

1. 생성자 주입 (권장)

    ```java
    private final MemberService memberService;

    @Autowired
    public MemberController(MemberService memberService) {
        this.memberService = memberService;
    }
    ```

2. setter 주입 (setter가 Public하게 노출되는 문제 발생)

    ```java
    public class MemberController {

        private MemberService memberService;

        @Autowired
        public void setMemberService(MemberService memberService) {
            this.memberService = memberService;
        }
    ```

3. 필드 주입 (별로 안 좋음 - 중간에 바꾸기 힘들다?)

    ```java
    public class MemberController {

        @Autowired private final MemberService memberService;
    ```

### TIP:

- Autowired를 통한 DI는, Spring이 관리하는 객체에서만 동작한다. 즉, 빈으로 등록하지 않고 직접 생성한 객체에서는 동작하지 않는다.