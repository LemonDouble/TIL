# Spring 핵심 원리 : 싱글톤 컨테이너

분류: Spring
작성일시: 2021년 7월 16일 오후 5:49

## 1. 웹 어플리케이션과 싱글톤

- 스프링은 태생이 기업용 온라인 서비스 기술을 지원하기 위해 탄생했다.
- 대부분의 스프링 어플리케이션은 웹 어플리케이션.
- 웹 어플리케이션은 보통 여러 고객이 동시에 요청을 한다.

![Spring%20%E1%84%92%E1%85%A2%E1%86%A8%E1%84%89%E1%85%B5%E1%86%B7%20%E1%84%8B%E1%85%AF%E1%86%AB%E1%84%85%E1%85%B5%20%E1%84%89%E1%85%B5%E1%86%BC%E1%84%80%E1%85%B3%E1%86%AF%E1%84%90%E1%85%A9%E1%86%AB%20%E1%84%8F%E1%85%A5%E1%86%AB%E1%84%90%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%82%E1%85%A5%20cb5b8cfedf28446fa5ce5a8fe9cf9df5/Untitled.png](https://github.com/LemonDouble/TIL/blob/main/spring/img/Untitled%2014.png)

- 직접 만든, Spring 없는 순수한 DI 컨테이너는 고객 요청시 새로 객체를 생성한다!
- 만약, 고객 트래픽이 초당 100이라면 초당 100개의 객체가 생성되고 소멸된다 → 메모리 소모가 심하다
- 해결 방법으론, 해당 객체가 딱 하나만 생성되고 공유되도록, 싱글톤 패턴을 사용한다.

- 순수한 DI 컨테이너일때, 다른 객체가 생성되는것을 확인하는 코드

    /test/hello.core/singleton/SingletonTest.java

```java
package singleton;

import hello.core.AppConfig;
import hello.core.member.MemberService;
import org.assertj.core.api.Assertions;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;

public class SingletonTest {
    @Test
    @DisplayName("스프링 없는 순수한 DI 컨테이너")
    void pureContiner(){
        AppConfig appConfig = new AppConfig();

        //1. 조회 : 호출할 때 마다 객체를 생성
        MemberService memberService1 = appConfig.memberService();

        //2. 조회 : 호출할 때 마다 객체를 생성
        MemberService memberService2 = appConfig.memberService();

        //참조값이 다른 것을 확인
        System.out.println("memberService1 = " + memberService1);
        System.out.println("memberService2 = " + memberService2);

        //MemberService1 =/= memberService2
        Assertions.assertThat(memberService1).isNotSameAs(memberService2);

    }
}
```

## 2. 싱글톤 패턴

- 클래스의 인스턴스가 딱 1개만 생성되는 것을 보장하는 디자인 패턴이다.
- 그래서 객체 인스턴스를 2개 이상 생성하지 못하도록 막아야 한다.

- 싱글톤 객체의 예시 코드 (/test/hello.core/singleton/SingletonService.class)

```java
package singleton;

public class SingletonService {

    //자기 자신을 내부에 private static final 선언, 단 하나만 생성된다.
    //JVM이 실행될 때 SingletonService 생성하고, instance 에 참조로 넣어 둔다.
    private static final SingletonService instance = new SingletonService();

    //SingletonService를 사용하려면 getInstance 메소드만을 통해야 한다.
    public static SingletonService getInstance() {
        return instance;
    }

    //생성자를 private로 만들면, 외부에서 new 키워드로 생성 불가하다.
		//(Compile Error)
    private SingletonService(){
    }

    public void logic(){
        System.out.println("싱글톤 객체 로직 호출");
    }

}
```

- 싱글톤 객체의 테스트 코드 ( 위의 SingletonTest.class에 추가)

```java
@Test
    @DisplayName("싱글톤 패턴을 적용한 객체 사용")
    void singletonServiceTest(){
        SingletonService singletonService1 = SingletonService.getInstance();
        SingletonService singletonService2 = SingletonService.getInstance();

        System.out.println("singletonService1 = " + singletonService1);
        System.out.println("singletonService2 = " + singletonService2);
        
        //Same : == (객체의 메모리 주소를 비교)
        //Equal : .equals 메소드 (객체의 실제 값을 비교, Memory는 다를 수 있음)
        Assertions.assertThat(singletonService1).isSameAs(singletonService1);
    }
```

- 싱글톤 패턴을 적용함으로써, 이미 만들어진 객체를 공유해 효율적으로 사용할 수 있지만, 다음과 같은 문제점을 가지고 있다.

- 싱글톤 패턴의 문제점
    1. 싱글톤 패턴을 구현하는데 코드가 많이 들어간다.
    2. 의존관계상 클라이언트가 구체 클래스에 의존한다 → DIP를 위반한다.
    3. 클라이언트가 구체 클래스에 의존해서 OCP 원칙을 위반할 가능성이 높다.
    4. 테스트하기 어렵다.
    5. 내부 속성을 변경하거나 초기화하기 어렵다.
    6. private 생성자로써 자식 클래스를 만들기 어렵다.
    7. 유연성이 떨어진다.
    8. 안티패턴으로 불리기도 한다.

- 추가 : isSameAs와 isEqualTo

```java
package singleton;

import org.assertj.core.api.Assertions;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;

public class equalssame {

    @Test
    @DisplayName("같은 값, 같은 주소 테스트")
    void SameAddress(){
        String msg1 = "msg";
        String msg2 = "msg";

        Assertions.assertThat(msg1).isSameAs(msg2); // 같은 주소이므로 true
        Assertions.assertThat(msg1).isEqualTo(msg2); // 같은 값 (msg) 이므로 true
    }

    @Test
    @DisplayName("같은 값, 다른 주소 테스트")
    void DiffAddress(){
        String msg1 = "msg";
        String msg2 = new String("msg");

        Assertions.assertThat(msg1).isNotSameAs(msg2); // 서로 다른 주소이므로 isNotSameAs
        Assertions.assertThat(msg1).isEqualTo(msg2); // 서로 같은 값이므로 isEqualTo
    }
}
```

## 3. 싱글톤 컨테이너

- 스프링 컨테이너는 싱글톤 패턴의 문제점을 해결하면서, 객체 인스턴스를 싱글톤으로 관리한다.

![Spring%20%E1%84%92%E1%85%A2%E1%86%A8%E1%84%89%E1%85%B5%E1%86%B7%20%E1%84%8B%E1%85%AF%E1%86%AB%E1%84%85%E1%85%B5%20%E1%84%89%E1%85%B5%E1%86%BC%E1%84%80%E1%85%B3%E1%86%AF%E1%84%90%E1%85%A9%E1%86%AB%20%E1%84%8F%E1%85%A5%E1%86%AB%E1%84%90%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%82%E1%85%A5%20cb5b8cfedf28446fa5ce5a8fe9cf9df5/Untitled%201.png](https://github.com/LemonDouble/TIL/blob/main/spring/img/Untitled%2015.png)

- 싱글톤 컨테이너
    - 스프링 컨테이너는 싱글톤 패턴을 적용하지 않아도, 객체 인스턴스를 싱글톤으로 관리한다.
    - 스프링 컨테이너는 싱글톤 컨테이너 역할을 한다.
    - 이렇게 싱글톤 객체를 생성/관리하는 기능을 싱글톤 레지스트리라 한다.
    - 스프링 컨테이너의 이런 기능을 통해 싱글턴 패턴의 단점을 해결하며 객체를 싱글톤으로 유지할 수 있다.
    - 싱글톤 패턴을 위해 지저분한 코드가 들어가지 않아도 된다.
    - DIP, OCP, 테스트, private 생성자로부터 자유롭게 싱글톤을 사용할 수 있다.

- SingletonTest에 추가

```java
@Test
    @DisplayName("스프링 컨테이너와 싱글톤")
    void springContainer(){

        //Spring 컨테이너 사용
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

        MemberService memberService1 = ac.getBean("memberService", MemberService.class);
        MemberService memberService2 = ac.getBean("memberService", MemberService.class);

        //참조값이 같은 것을 확인
        System.out.println("memberService1 = " + memberService1);
        System.out.println("memberService2 = " + memberService2);

        //MemberService1 == memberService2
        Assertions.assertThat(memberService1).isSameAs(memberService2);
    }
```

## 4. 싱글톤 방식의 주의점

- 싱글톤 패턴이든, Spring같은 컨테이너를 사용하던, 객체 인스턴스를 하나만 생성해서 공유하는 싱글톤 방식은 여러 클라이언트가 하나의 같은 객체 인스턴스를 공유하기 때문에, 싱글톤 객체는 상태를 유지(Stateful) 하게 설계하면 안 된다!
- 따라서, 무상태 (Stateless)하게 설계해야 한다!
    - 특정 클라이언트에 의존적인 필드가 있으면 안 된다.
    - 특정 클라이언트가 값을 변경할 수 있는 필드가 있으면 안 된다.
    - 가능하다면, 읽기만 가능해야 한다.
    - 필드 대신 자바에서 공유되지 않는 지역 변수, 파라미터, ThreadLocal 드을 사용해야 한다.

- Spring Bean의 필드에 공유 값을 설정하면 정말 큰 장애가 발생할  수 있다!!!

- Stateful한 서비스의 예시
- test/../singleton/StatefulService.class

```java
package singleton;

public class StatefulService {

    private int price; // 상태를 유지하는 필드

    public void order(String name, int price){
        System.out.println("name = " + name + " price = " + price);
        this.price = price;
    }

    public int getPrice(){
        return price;
    }
}
```

- test/../singleton/StatefulServiceTest.class

```java
package singleton;

import org.assertj.core.api.Assertions;
import org.junit.jupiter.api.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;

import static org.assertj.core.api.Assertions.*;

public class StatefulServiceTest {

    @Test
    void statefulServiceSingleton(){

        ApplicationContext ac = new AnnotationConfigApplicationContext(TestConfig.class);

        StatefulService statefulService1 = ac.getBean(StatefulService.class);
        StatefulService statefulService2 = ac.getBean(StatefulService.class);

        //원래 Thread 사용해야 하지만, 간단한 예시로 대체

        //ThreadA : A 사용자가 10000원 주문
        statefulService1.order("userA", 10000);

        //ThreadB : B 사용자가 20000원 주문
        statefulService2.order("userB", 20000);

        //ThreadA : 사용자A 주문 금액 조회
        int price = statefulService1.getPrice();

        //10000원이 아니라 20000원이 나온다!
        System.out.println("price = " + price);

        assertThat(statefulService1.getPrice()).isEqualTo(20000);
    }

    static class TestConfig{
        @Bean
        public StatefulService statefulService(){
            return new StatefulService();
        }
    }
}
```

### >> 스프링 빈은 항상 Stateless하게 설계해야 한다는 것을 기억하자!! <<

## 5. Configuration과 싱글톤

- Appconfig.class에서

```java
//설정 정보 Annotation
@Configuration
public class AppConfig {

    //각 설정 정보를 Spring Container에 등록
    @Bean
    public MemberService memberService(){
        return new MemberServiceImpl(memberRepository());
    }

    @Bean
    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }

    @Bean
    public OrderService orderService(){
        return new OrderServiceImpl(memberRepository(),  discountPolicy());
    }

    @Bean
    public DiscountPolicy discountPolicy(){
        return new RateDiscountPolicy();
    }

}
```

1. memberService Bean을 등록할 때, memberRepository()를 호출 → memberRepository는 new MemoryMemberRepository를 호출
2. 이후, orderService Bean을 등록할 때, memberRepository()를 호출 → memberRepository는 new MemoryMemberRepository를 호출

결과적으로, 서로 다른 두개의 MemoryMemberRepository가 생성되며 싱글톤이 깨지는 것 처럼 보인다.

- 검증 코드 추가, test/.../singleton/ConfigurationSingletonTest.class

```java
package singleton;

import hello.core.AppConfig;
import hello.core.member.MemberRepository;
import hello.core.member.MemberServiceImpl;
import hello.core.order.OrderServiceImpl;
import org.assertj.core.api.Assertions;
import org.junit.jupiter.api.Test;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class ConfigurationSingletonTest {

    @Test
    void configurationTest(){
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

        MemberServiceImpl memberService = ac.getBean("memberService", MemberServiceImpl.class);
        OrderServiceImpl orderService = ac.getBean("orderService", OrderServiceImpl.class);
        MemberRepository memberRepository = ac.getBean("memberRepository", MemberRepository.class);

        MemberRepository memberRepository1 = memberService.getMemberRepository();
        MemberRepository memberRepository2 = orderService.getMemberRepository();

        //셋 다 같은 인스턴스!
        System.out.println("memberRepository => " + memberRepository);
        System.out.println("memberService => " + memberRepository1);
        System.out.println("orderService => " + memberRepository2);

        Assertions.assertThat(memberService.getMemberRepository()).isSameAs(memberRepository);
        Assertions.assertThat(orderService.getMemberRepository()).isSameAs(memberRepository);
    }
}
```

- Appconfig.class에 call Log 추가

```java
package hello.core;

import hello.core.discount.DiscountPolicy;
import hello.core.discount.RateDiscountPolicy;
import hello.core.member.MemberRepository;
import hello.core.member.MemberService;
import hello.core.member.MemberServiceImpl;
import hello.core.member.MemoryMemberRepository;
import hello.core.order.OrderService;
import hello.core.order.OrderServiceImpl;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AppConfig {

    @Bean
    public MemberService memberService(){
        System.out.println("AppConfig.memberService");
        return new MemberServiceImpl(memberRepository());
    }

    @Bean
    public MemberRepository memberRepository() {
        System.out.println("AppConfig.memberRepository");
        return new MemoryMemberRepository();
    }

    @Bean
    public OrderService orderService(){
        System.out.println("AppConfig.orderService");
        return new OrderServiceImpl(memberRepository(),  discountPolicy());
    }

    @Bean
    public DiscountPolicy discountPolicy(){
        return new RateDiscountPolicy();
    }

}
```

출력 결과 : AppConfig.memberService → AppConfig.memberRepository → AppConfig.orderService

중복 호출이 되지 않고 한 번만 호출된다!

## 6. Configuration과 바이트코드 조작의 마법

- 스프링 컨테이너는 싱글톤 레지스트리, 따라서 Spring Bean은 싱글톤이 되도록 보장해줘야 한다.
- 하지만 원래 Appconfig.class를 봤을 때, return new MemberRepository; 는 3번 호출되는것이 맞다.
- 그래서 Spring은 바이트코드를 조작하는 라이브러리를 사용한다.
- 즉, AppConfig.class를 조작한다!

- singleton/ConfigurationSingletonTest에 다음 test 추가

```java
@Test
    void configurationDeep(){

        ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
        //위와 같이 등록시, AppConfig.class도 Spring Bean으로 등록이 된다!
        AppConfig bean = ac.getBean(AppConfig.class);

        //출력 : bean.getClass() = class hello.core.AppConfig$$EnhancerBySpringCGLIB$$80db713f
        System.out.println("bean.getClass() = " + bean.getClass());

    }
```

→ AppConfig.class는, 내가 등록한 클래스가 아니라 CGLIB이란 바이트코드 조작 라이브러리를 사용해 Appconfig.class를 상속받은 다른 클래스를 만들고, 그 클래스를 Spring Bean으로 등록한 것이다!

![Spring%20%E1%84%92%E1%85%A2%E1%86%A8%E1%84%89%E1%85%B5%E1%86%B7%20%E1%84%8B%E1%85%AF%E1%86%AB%E1%84%85%E1%85%B5%20%E1%84%89%E1%85%B5%E1%86%BC%E1%84%80%E1%85%B3%E1%86%AF%E1%84%90%E1%85%A9%E1%86%AB%20%E1%84%8F%E1%85%A5%E1%86%AB%E1%84%90%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%82%E1%85%A5%20cb5b8cfedf28446fa5ce5a8fe9cf9df5/Untitled%202.png](https://github.com/LemonDouble/TIL/blob/main/spring/img/Untitled%2016.png)

- Spring이 바이트코드를 조작한 다른 클래스가, 해당 클래스가 싱글톤이 보장되도록 해 준다.

- 해당 AppConfig@CGLIB 예상 코드 ( 실제로는 훨씬 더 복잡하다! )

```java
@Bean
public MemberRepository memberRepository() {
	if (memoryMemberRepository가 이미 스프링 컨테이너에 등록되어 있으면?) {
		return 스프링 컨테이너에서 찾아서 반환;
	} else { //스프링 컨테이너에 없으면
	기존 로직을 호출해서 MemoryMemberRepository를 생성하고 스프링 컨테이너에 등록
		return 반환
	}
}
```

→ 예시와 같이, 스프링 빈이 존재하면 존재하는 스프링 빈은 반환, 스프링 빈이 없으면 스프링 빈으로 등록하고 반환하는 코드를 동적으로 만든다.

참고 ) AppConfig@CGLIB은 AppConfig의 자식 타입이므로, AppConfig 타입으로 조회할 수 있다.

- 만약 @Configuration을 적용하지 않는다면?
    - CGLIB을 사용하지 않으므로, 싱글톤을 보장하지 않는다.
    - 내가 작성한 AppConfig가, 코드 변환 없이 그대로 Spring Bean으로 등록된다.

    ```
    call AppConfig.memberService
    call AppConfig.memberRepository
    call AppConfig.orderService
    call AppConfig.memberRepository
    call AppConfig.memberRepository
    ```

    - 우리가 예측한 것 대로, memberRepository가 세번 호출되고, 각 인스턴스는 서로 다른 memberRepository이다.
