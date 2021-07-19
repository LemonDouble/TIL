# Spring 핵심 원리 : 의존관계 자동 주입

분류: Spring
작성일시: 2021년 7월 18일 오후 5:03

## 1. 다양한 의존관계 주입 방법

- 크게 4가지 방법이 있다
    - 생성자 주입 (권장!)
    - 수정자 주입(setter 주입)
    - 필드 주입
    - 일반 메서드 주입

1. 생성자 주입
    - 이름 그대로 생성자를 통해 의존 관계를 주입하는 방법

    - 특징)
        - 생성자 호출 시점에 단 한번만 호출되는 것이 보장된다.
        - >불변, 필수< 의존관계에 사용

            (만약, Setting 후 값이 바뀌면 안 되는 값이나 내용이 있다면, 생성자에서 설정을 마치고 Getter/Setter 메소드를 정의하지 말자)

        ```java
        @Component
        public class OrderServiceImpl implements OrderService {
            
            //Final : 값이 없으면 Error, 반드시 값을 넣어줘야 함을 언어로써 표현
            private final MemberRepository memberRepository;
            private final DiscountPolicy discountPolicy;

            @Autowired
            public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
                this.memberRepository = memberRepository;
                this.discountPolicy = discountPolicy;
            }
        }
        ```

        - 중요 : 생성자가 단 하나만 있다면, @Autowired 생략해도 자동 주입된다. (Spring Bean인 경우)

        ```java
        @Component
        public class OrderServiceImpl implements OrderService {
            
            private final MemberRepository memberRepository;
            private final DiscountPolicy discountPolicy;

            // @Autowired 생략되어도 OK
            public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
                this.memberRepository = memberRepository;
                this.discountPolicy = discountPolicy;
            }
        }

        @Component
        public class OrderServiceImpl implements OrderService {
            
            private final MemberRepository memberRepository;
            private final DiscountPolicy discountPolicy;

            // 생성자가 2개 이상이므로, @Autowired 생략하면 안 됨
        		@Autowired
            public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
                this.memberRepository = memberRepository;
                this.discountPolicy = discountPolicy;
            }

            public OrderServiceImpl(DiscountPolicy discountPolicy) {
                this.memberRepository = memberRepository;
                this.discountPolicy = discountPolicy;
            }
        }
        ```

1. 수정자 주입(setter 주입)
    - setter라 불리는 필드의 값을 변경하는 수정자 메소드를 통해 의존관계를 주입하는 방법

    - 특징)
        - >선택, 변경< 가능성이 있는 의존관계에 사용
        - 자바 빈 프로퍼티 규약의 수정자 메소드 방식을 사용하는 방법

        ```java
        @Component
        public class OrderServiceImpl implements OrderService {
            
            private MemberRepository memberRepository;
            private DiscountPolicy discountPolicy;

            @Autowired
            public void setMemberRepository(MemberRepository memberRepository) {
                this.memberRepository = memberRepository;
            }

            @Autowired
            public void setDiscountPolicy(DiscountPolicy discountPolicy) {
                this.discountPolicy = discountPolicy;
            }
        ```

### Spring은 Bean 등록 → 의존관계 주입 순으로 LifeCycle을 가진다.

- 이때, Bean을 등록할 땐 생성자를 호출되어야 하므로, 생성자 주입은 Bean 등록 시에 호출된다.
- Setter 주입 시에는, 의존관계 주입 시점에 호출된다.
- 따라서, 생성자 주입은 Setter 주입보다 항상 먼저 호출된다.

1. 필드 주입 (사용하지 말것)
    - 이름 그대로 필드에 바로 주입하는 방식이다.

    - 특징)
        - 외부에서 변경이 불가능해, 테스트하기 힘들다는 치명적인 단점이 있다.
        - DI 프레임워크가 없다면 아무것도 할 수 없다.
        - 사용하지 말자! (@Configuration 같은 곳에서, 특수한 용도로만 사용한다)

    ```java
    @Component
    public class OrderServiceImpl implements OrderService {
        
        @Autowired private MemberRepository memberRepository;
        @Autowired private DiscountPolicy discountPolicy;
    }
    ```

1. 메서드 주입
    - 일반 메서드를 통해서 주입받을 수 있다

    - 특징)
        - 한번에 여러 필드를 주입받을 수 있다.
        - 일반적으로 잘 사용하지 않는다.

        ```java
        @Component
        public class OrderServiceImpl implements OrderService {
            
            private MemberRepository memberRepository;
            private DiscountPolicy discountPolicy;

            @Autowired
            public void init(MemberRepository memberRepository, DiscountPolicy discountPolicy){
                this.memberRepository = memberRepository;
                this.discountPolicy = discountPolicy;
            }
        ```

## 2. 옵션 처리

- 주입할 스프링 빈이 없어도 동작해야 할 때가 있다.
- 예를 들어, Spring Bean이 없다면 기본 로직으로 동작하게 하거나, 일부 빈이 Optional이거나..

- 하지만 @Autowired의 required 옵션의 기본값은 true, 즉 자동 주입 대상이 없으면 오류가 발생한다.

- 자동 주입을 옵션으로 처리하는 방법
    - @Autowired(required = false) : 자동 주입할 대상 없으면 Setter 자체가 호출이 안 된다
    - org.springframework.lang.@Nullable : 자동 주입할 대상 없으면 null이 입력된다.
    - Optional <> : 자동 주입할 대상이 없으면 Optional.empty가 입력된다

- Test/../autowired/AutowireTest.class

```java
package hello.core.autowired;

import hello.core.member.Member;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.lang.Nullable;

import java.util.Optional;

public class AutowiredTest {

    @Test
    void AutowiredOption(){
            ApplicationContext ac = new AnnotationConfigApplicationContext(TestBean.class);
    }

    static class TestBean{

        //setNoBean 메서드 자체가 호출되지 않음
        @Autowired(required = false)
        public void setNoBean1(Member noBean1){
            System.out.println("noBean1 = " + noBean1);
        }
        
        //noBean2 = null
        @Autowired
        public void setNoBean2(@Nullable Member noBean2){
            System.out.println("noBean2 = " + noBean2);
        }
        
        //noBean3 = Optional.empty
        @Autowired
        public void setNoBean3(Optional<Member> noBean3){
            System.out.println("noBean3 = " + noBean3);
        }

    }
}
```

## 3. 생성자 주입을 사용하자!

- 대부분은 생성자 주입을 권장하는데, 이유는 다음과 같다.
    - 대부분의 의존관계 주입은 한번 일어나면 어플리케이션 종료 시점까지 의존관게를 변경할 일이 없다.
    - 오히려, 대부분의 의존관계는 어플리케이션 종료 전까지 변하면 안 된다.
    - 수정자 주입을 사용하면 set Method를 public으로 열어둬야 한다.
    - 누군가 실수로 변경할 수도 있고, 변경하면 안 되는 메서드를 열어두는 것 자체가 좋은 설계 방식이 아니다.
    - 생성자 주입은 객체 생성 시 딱 1회만 호출되므로 이후 호출되지 않음이 보장된다. 따라서 불변하도록 설계할 수 있다.

- 누락
    - 프레임워크 없이 순수한 자바 코드를 단위 테스트 하는 경우가 많다.
    - 다음과 같이 수정자 주입을 사용하는 경우 (orderServiceImpl.class)

    ```java
    @Component
    public class OrderServiceImpl implements OrderService {

        private MemberRepository memberRepository;
        private DiscountPolicy discountPolicy;

        //Setter 주입
        @Autowired
        public void setMemberRepository(MemberRepository memberRepository) {
            this.memberRepository = memberRepository;
        }

        @Autowired
        public void setDiscountPolicy(DiscountPolicy discountPolicy) {
            this.discountPolicy = discountPolicy;
        }
    ```

- 다음과 같이 테스트를 수행하면 NullPointExceoption이 발생한다.

    (memberRepository, discountPolicy 주입을 누락했기 때문에.)

    하지만 생성자가 new OrderServiceImpl() 이므로, 무엇이 누락되었는지 알려면 직접 OrderServiceImpl.class를 확인해야 한다.

```java
package hello.core.order;

import org.junit.jupiter.api.Test;

public class OrderServiceImplTest {

    @Test
    void createOrder(){
        OrderServiceImpl orderService = new OrderServiceImpl();
        orderService.createOrder(1L, "itemA", 10000);

    }
}
```

- 하지만 생성자 주입을 사용할 경우, 컴파일 오류가 발생한다.

    (생성자의 인자로 MemoryMemberRepository, DiscountPolicy가 누락됨)

    따라서 IDE에서 어떤 값을 주입해야 하는지 빠르게 알 수 있다.

- final 키워드
    - 생성자 주입을 사용하면 필드에 final 키워드를 사용할 수 있다.
    - 따라서,필드에 값이 설정되지 않는 오류를 컴파일 시점에 막아준다.

```java
@Component
public class OrderServiceImpl implements OrderService {

    //final 키워드 사용 가능!
    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;

    @Autowired
    public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }
```

- 값이 세팅되지 않은 경우, java: variable XXX might not have been initialized 오류가 발생한다.
- 컴파일 오류는 빠르고, 좋은 오류다!

- 요약)
    - 생성자 주입 방법을 선택하면, 프레임워크에 의지하지 않고 순수한 자바 언어의 특징을 잘 살릴 수 있다.
    - 기본으로 생성자 주입을 사용하고, 필수 값이 아닌 경우는 수정자 주입 방식을 옵션으로 부여하면 된다.

## 4. 롬복과 최신 트랜드

- 롬복(Lombok : [https://projectlombok.org/](https://projectlombok.org/)) : 반복해서 작성해야 하는 Constructor, Getter, Setter 등을 자동으로 작성하도록 도와주는 라이브러리

- build.gradle에 lombok 관련 내용 추가

```java
plugins {
	id 'org.springframework.boot' version '2.5.2'
	id 'io.spring.dependency-management' version '1.0.11.RELEASE'
	id 'java'
	id 'jacoco'
}

group = 'hello'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11'

//lombok 설정 추가 시작
configurations {
	compileOnly {
		extendsFrom annotationProcessor
	}
}
//lombok 설정 추가 끝

repositories {
	mavenCentral()
}

dependencies {
	implementation 'org.springframework.boot:spring-boot-starter'

	//lombok 라이브러리 추가 시작
	compileOnly 'org.projectlombok:lombok'
	annotationProcessor 'org.projectlombok:lombok'
	testCompileOnly 'org.projectlombok:lombok'
	testAnnotationProcessor 'org.projectlombok:lombok'
	//lombok 라이브러리 추가 끝
	
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

test {
	useJUnitPlatform()
}
```

- File→Settings→Plugin → lombok 설치 ( 2020.3부턴 Intellij 기본 지원 )
- File→Settings→annotation process 검색 → Compiler/Annotation Processors 누른 후 Enable annotation processing Enable

- 이후 다음과 같이, Annotation을 통해 간단하게 Getter, Setter 등을 만들 수 있다.

```java
package hello.core;

import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public class HelloLombok {
    private String name;
    private int age;

    public static void main(String[] args){
        HelloLombok helloLombok = new HelloLombok();
        helloLombok.setName("Hello, Lombok!");

        String name = helloLombok.getName();
        System.out.println("name = " + name);
    }
}
```

- Lombok의 RequiredArgsConstructor를 통해 다음처럼 Constructor를 자동 생성할 수 있다.
- 이 때, 생성자가 하나 뿐인 경우 @Autowired를 생략해도 되므로 결과적으로

```java
@Component
//이 Annotation은 final 키워드가 붙은 변수들에게
@RequiredArgsConstructor
public class OrderServiceImpl implements OrderService {

    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;

    @Autowired
    //아래와 같은 Constructor를 만들어 준다!
    public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }
```

- 아래와 같이 간단하게 만들 수 있다!

```java
@Component
@RequiredArgsConstructor
public class OrderServiceImpl implements OrderService {

    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;
}
```

## 5. 조회 빈이 2개 이상일 때 생기는 문제

- 만약 다음과 같은  OrderServiceImpl.class가 있을 때

```java
@Component
public class OrderServiceImpl implements OrderService {

    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;

    @Autowired
    public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }
```

- 생성자에서 DI를 할  때, 타입으로 조회하므로 다음 코드와 유사하게 동작한다.

```java
this.discountPolicy = ac.getBean(DiscountPolicy.class)
```

- 하지만 이때, 타입으로 조회할 시, 같은 타입의 빈이 2개 이상이라면 오류가 발생한다.

    만약 DiscountPolicy의 하위 타입인 FixDiscountPolicy, RateDiscountPolicy 둘 다 빈으로 등록되어 있다면 " NoUniqueBeanDefinitionException " 에러가 발생한다.

    ( Fix, Rate중 무엇을 등록해야 하는지 알 수 없다.)

## 6. @Autowired 필드명, @Qualifier, @Primary (중복 해결)

- 조회 대상 빈이 2개 이상일 때 해결 방법은 다음과 같다
    1. Autowired 필드 명 매칭
    2. Qualifier → Qualifier끼리 매칭 → 빈 이름 매칭
    3. Primary 사용

- Autowired 필드 명 매칭
    - @Autowired는 타입 매칭을 먼저 시도하고, 이때 여러 빈이 있다면 필드 이름, 파라미터 이름으로 빈 이름을 추가로 매칭한다.
    - 다음 생성자의 경우, DiscountPolicy 타입이 Fix, Rate 두개이므로 NoUniqueBeanDefinitionException  에러가 발생한다. 이때 파라미터를 수정해

    ```java
    @Autowired
        public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
            this.memberRepository = memberRepository;
            this.discountPolicy = discountPolicy;
        }
    ```

    - 다음과 같이 바꿀 경우,

        DiscountPolicy 타입을 찾음 → Fix / Rate, 2개 이상의 타입이 나옴 → 파라미터를 이용해 rateDiscountPolicy와 매칭된다.

    ```java
    @Autowired
        public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy rateDiscountPolicy) {
            this.memberRepository = memberRepository;
            this.discountPolicy = rateDiscountPolicy;
        }
    ```

- Autowired 매칭 정리
    1. 타입 매칭
    2. 타입 매칭의 결과가 2개 이상일 때, 필드 명, 파라미터 명으로 Bean 이름 매칭

- 참고 ) Spring은 타입 매칭 시 자식 타입도 모두 가져온다. ( DiscountPolicy로 매칭 시 Fix, Rate 전부 가져온다)

---

- Qualifier 사용
    - Qualifier는 추가 구분자를 붙여주는 방법이다.
    - 주입시 추가적인 방법을 제공하는 것이지, 빈 이름 자체를 변경하는 것은 아니다.

예시)

- RateDiscountPolicy에 Qualifier "mainDiscountPolicy" 를 추가한다.

```java
@Component
@Qualifier("mainDiscountPolicy")
public class RateDiscountPolicy implements DiscountPolicy {
```

- OrderServiceImpl에서, Qualifier를 추가해 파라미터로 넘겨주면, 해당 Qualifier와 매칭되는 클래스를 주입한다.

```java
@Component
public class OrderServiceImpl implements OrderService {

    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;

    @Autowired
    public OrderServiceImpl(MemberRepository memberRepository, @Qualifier("mainDiscountPolicy") DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }
```

- 생성자 뿐 아니라, Setter, Bean 수동 등록 시에도 동일하게 사용할 수 있다.

- 만약 , Spring이 "mainDiscountPolicy" 라는 Qualifier를 못 찾았다면 mainDiscountPolicy라는 이름의 Spring bean을 찾지만, Qualifier는 Qualifier를 찾는 용도로만 사용하는 것이 좋다.

- Qualifier 매칭 정리
    1. Qualifier끼리 매칭
    2. Qualifier 없다면, 빈 이름 매칭
    3. NoSuchBeanDefinitionException 예외 발생

---

- Primary 사용
    - Primary는 우선순위를 정하는 방법이다. @Autowired 시에 여러 빈이 매칭되면, Primary가 우선권을 가진다.
    - 다음과 같이 RateDiscountPolicy에 @Primary가 있다면, Fix를 무시하고 Rate와 매칭된다.

    ```java
    @Component
    @Primary
    public class RateDiscountPolicy implements DiscountPolicy {
    ..
    }
    ```

---

- Primary, Qualifier의 사용

    만약 주로 사용하는 메인 DB와, 아주 가끔 사용하는 Sub DB가 있다고 가정했을 때,

    Main DB에는 @Primary를 사용하여 편리하게 조회하고,

    Sub DB에는 @Qualifier를 지정하여 명시적으로 조회하는 식으로 사용할 수 있다.

- 우선순위

    Primary와 Qualifier가 겹칠 경우, Qualifier가 우선순위를 가진다.

    (Qualifier는 직접 수동으로 지정, Pirmary는 기본값으로 사용이므로, 자동보다는 수동이, 넓은 범위보다는 좁은 범위의 선택권이 항상 우선순위를 가진다.)

## 7. 어노테이션 직접 만들기

- @Qualifier("mainDiscountPolicy") 와 같이 적으면, 컴파일시 타입 체크가 되지 않는다.
- 어노테이션을 만들어 이런 문제를 해결할 수 있다.

- main/hello.core/annotation 패키지 만든 후, MainDiscountPolicy annotation 추가

```java
package hello.core.annotation;

import org.springframework.beans.factory.annotation.Qualifier;

import java.lang.annotation.*;

//Qualifier Annotation에서 가져온 내용
//Intellij에서 F4 누르면 해당 클래스로 이동한다
@Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER, ElementType.TYPE, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Qualifier("mainDiscountPolicy")
public @interface MainDiscountPolicy {
}
```

- 이후, 사용할 곳에 Annotation 추가

- RateDiscountPolicy.class

```java
@MainDiscountPolicy
public class RateDiscountPolicy implements DiscountPolicy {
...
}
```

- OrderServiceImpl.class

```java
@Autowired
    public OrderServiceImpl(MemberRepository memberRepository, @MainDiscountPolicy DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }
```

- 참고 )

    Annotation에는 상속이라는 개념이 없다. Annotation 여러 개를 모아서 사용하는 것은 Spring이 지원하는 기능이다.

    다른 Annotation들도 조합하여 사용할 수 있지만, 뚜렷한 목적 없이 Annotation을 재정의하고 사는 것은 유지보수에 오히려 더 혼란만 가중할 수 있다.

## 8. 조회한 빈이 모두 필요할 때, List, Map

- 만약 할인 서비스를 제공할 때, 유저가 어떤 할인을 선택할지 정하는 경우
- 전략 패턴(Strategy Pattern) 를 쉽게 구현할 수 있다!

- test/.../autowired/allbean/AllBeanTest.class

```java
package hello.core.autowired.allbean;

import hello.core.AutoAppConfig;
import hello.core.discount.DiscountPolicy;
import hello.core.member.Grade;
import hello.core.member.Member;
import org.assertj.core.api.Assertions;
import org.junit.jupiter.api.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

import java.util.List;
import java.util.Map;

public class AllBeanTest {

    @Test
    void findAllBean(){
        //AutoAppConfig 등록 후, DiscountService 등록한다.
        ApplicationContext ac = new AnnotationConfigApplicationContext(AutoAppConfig.class, DiscountService.class);

        //DiscountService 선언해서
        DiscountService discountService = ac.getBean(DiscountService.class);
        Member member = new Member(1L, "userA", Grade.VIP);
        //DiscountPolicy 이름으로 match 한다
        int discountPrice = discountService.discount(member,10000, "fixDiscountPolicy");

        Assertions.assertThat(discountService).isInstanceOf(DiscountService.class);
        Assertions.assertThat(discountPrice).isEqualTo(1000);

        int rateDiscountPrice = discountService.discount(member, 20000, "rateDiscountPolicy");

        Assertions.assertThat(rateDiscountPrice).isEqualTo(2000);
    }

    static class DiscountService{
        private final Map<String, DiscountPolicy> policyMap;
        //List로도 받을 수 있음을 보기 위해 받음 (여기서 사용은 X)
        private final List<DiscountPolicy> policies;

        //모든 DisocuntPolicy를 주입받는다
        public DiscountService(Map<String, DiscountPolicy> policyMap, List<DiscountPolicy> policies) {
            this.policyMap = policyMap;
            this.policies = policies;

            System.out.println("policyMap = " + policyMap);
            System.out.println("policies = " + policies);
        }

        public int discount(Member member, int price, String discountCode) {
            //이름으로 할인 정책을 찾는다
            DiscountPolicy discountPolicy = policyMap.get(discountCode);
            //해당 할인 정책의 할인 값을 반납한다.
            return discountPolicy.discount(member,price);
        }
    }
}
```

## 9. 자동/수동 빈 등록의 올바른 실무 운영 기준

- 기본적으론, 편리한 자동 기능을 기본으로 사용한다.
    - @Component, @Controller, @Service, @Repository 처럼 계층에 맞춰 일반적인 어플리케이션 로직을 자동으로 사용하도록 지원
    - (개발자 입장으로) 자동 등록에 비해 직접 수동 등록하는 것은ㅇ 번거롭다.
    - 또한 Bean 개수가 많아지면 설정 정보 관리도 부담이 된다.
    - 결정적으로, OCP, DIP를 자동 빈 등록을 통해서도 지킬 수 있다.

        (수동 등록이나, 자동 등록이나 

- 그러면 수동 빈 등록은 언제 사용하면 좋을까?
    - 어플리케이션은 크게, 업무 로직과 기술 지원 로직으로 나눌 수 있다.
        - 업무 로직 빈 : 웹을 지원하는 컨트롤러, 핵심 비즈니스 로직이 있는 서비스. 데이터 계층의 로직을 처리하는 리포지토리(DB, JPA..) 등이 전부 업무 로직이다. 보통 비즈니스 요구사항을 개발할 때 추가/변경된다
        - 기술 지원 빈 : 기술적인 문제나 공통 관심사(AOP) 를 처리할 때 주로 사용된다. DB 연결, 공통 로그 처리처럼 업무 로직을 지원하기 위한 하부 기술이나 공통 기술들이다.

    - 업무 로직은 숫자도 매우 많고, 한번 개발하면 컨트롤러, 서비스, 리포지토리처럼 어느정도 유사한 패턴이 있음
    - 이런 경우, 자동 기능을 적극 사용하는게 좋음. 문제가 발생해도 어떤 곳에서 문제가 발생했는지 명확하게 파악하기 쉬움

    - 기술 지원 로직은 업무 로직과 비교해 수가 매우 적고, 보통 어플리케이션 전반에 광범위하게 영향을 미친다.
    - 또한 기술 지원 로직은 문제가 발생했을 때 파악하기 쉽지 않다.
    - 따라서 기술 지원 로직은 가급적 수동 빈 등록을 사용해 명확하게 드러내는 것이 좋다.

- 비즈니스 로직 중 다형성을 적극 활용할 때 (수동 빈 등록, 혹은 Package로 묶기)
    - 방금과 같은 DiscountService가 의존관계 자동 주입으로 Map<String, DiscountPolicy>에 주입받는 상황
    - 어떤 빈들이 주입될지, 각 빈들의 이름이 무엇일지 자동 등록을 통하면 알기 쉽지 않다.
    - 이런 경우, 수동 빈으로 등록하거나 "특정 패키지에 같이 묶어두는 것" 이 좋다.
    - 핵심은 한번에 이해가 되도록 하는 것

    ```java
    @Configuration
    public class DiscountPolicyConfig {
        @Bean
        public DiscountPolicy rateDiscountPolicy() {
            return new RateDiscountPolicy();
        }
        @Bean
        public DiscountPolicy fixDiscountPolicy() {
            return new FixDiscountPolicy();
        }
    }
    ```

    - 다음과 같이 별도의 설정 정보를 만들고 수동으로 등록하면, 한 눈에 어떤 빈이 들어올지 알 수 있다.

- 스프링/스프링 부트가 자동으로 등록하는 빈들은 별도로 관리하지 않는다.
    - 스프링 자체를 잘 이해하고, 스프링의 의도대로 잘 사용하는 것이 중요