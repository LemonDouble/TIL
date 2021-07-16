# Spring 핵심 원리 : 스프링 컨테이너와 스프링 빈

분류: Spring
작성일시: 2021년 7월 15일 오후 6:40

## 1. 스프링 컨테이너 생성

```java
ApplicationContext applicationContext = new AnnotationConfigApplicationContext();
```

- 이때 ApplicationContext를 스프링 컨테이너라고 한다.
- ApplictaionContext는 인터페이스다.
- 컨테이너는 XML, 어노테이션 기반의 자바 설정 기반(Appconfig.class) 로 만들 수도 있다. (주로 Annotation 기반의 자바 설정 기반을 주로 사용)

- 이 때, AnnotationConfigApplicationContext는 ApplictaionContext의 구현체이다.

// 정확히는, Spring Container는 BeanFactory, ApplicationContext 두 가지가 있다.

// 하지만, BeanFactory를 직접 사용하는 경우는 거의 없으므로, 일반적으론 ApplicationContext를

// Spring Container라고 한다.

- Spring 컨테이너의 생성 과정

    // 기억할 것 : Bean 이름은 항상 다른 이름을 부여해야 한다! 

    // 같은 이름을 부여하면 다른 빈이 무시되거나, 기존 빈을 덮어버리거나 할 수 있다

![Spring%20%E1%84%92%E1%85%A2%E1%86%A8%E1%84%89%E1%85%B5%E1%86%B7%20%E1%84%8B%E1%85%AF%E1%86%AB%E1%84%85%E1%85%B5%20%E1%84%89%E1%85%B3%E1%84%91%E1%85%B3%E1%84%85%E1%85%B5%E1%86%BC%20%E1%84%8F%E1%85%A5%E1%86%AB%E1%84%90%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%82%E1%85%A5%E1%84%8B%E1%85%AA%20%E1%84%89%E1%85%B3%E1%84%91%E1%85%B3%E1%84%85%E1%85%B5%E1%86%BC%20%E1%84%87%E1%85%B5%201622caf205e5420eab4a8ed0ec3b075f/Untitled.png](Spring%20%E1%84%92%E1%85%A2%E1%86%A8%E1%84%89%E1%85%B5%E1%86%B7%20%E1%84%8B%E1%85%AF%E1%86%AB%E1%84%85%E1%85%B5%20%E1%84%89%E1%85%B3%E1%84%91%E1%85%B3%E1%84%85%E1%85%B5%E1%86%BC%20%E1%84%8F%E1%85%A5%E1%86%AB%E1%84%90%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%82%E1%85%A5%E1%84%8B%E1%85%AA%20%E1%84%89%E1%85%B3%E1%84%91%E1%85%B3%E1%84%85%E1%85%B5%E1%86%BC%20%E1%84%87%E1%85%B5%201622caf205e5420eab4a8ed0ec3b075f/Untitled.png)

## 2. 컨테이너에 등록된 모든 빈 조회

- test/hello.core/beanfind package 만들고, ApplicationContextInfoTest.class 추가

```java
package hello.core.beanfind;

import hello.core.AppConfig;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.config.BeanDefinition;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class ApplicationContextInfoTest {

    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

    @Test
    @DisplayName("모든 Bean 출력하기")
    void findAllBean(){
        String[] beanDefinitionNames = ac.getBeanDefinitionNames();

        //list 선언 후 iter + 엔터
        for (String beanDefinitionName : beanDefinitionNames) {
            Object bean = ac.getBean(beanDefinitionName);
            //soutv + 엔터 : 변수명 찍어줌
            System.out.println("name = " + beanDefinitionName + "object = " + bean);
        }
    }

    @Test
    @DisplayName("모든 Application 출력하기")
    void findApplicationBean(){
        String[] beanDefinitionNames = ac.getBeanDefinitionNames();

        for (String beanDefinitionName : beanDefinitionNames) {
            //BeanDefinition : Bean의 Meta data
            BeanDefinition beanDefinition = ac.getBeanDefinition(beanDefinitionName);

            // Role ROLE_INFRASTRUCTURE = 스프링이 내부에서 사용하는 빈
            // Role ROLE_APPLICATION = 직접 등록한 애플리케이션 빈
            if (beanDefinition.getRole() == BeanDefinition.ROLE_APPLICATION){
                Object bean = ac.getBean(beanDefinitionName);
                System.out.println("name = " + beanDefinitionName + "object = " + bean);
            }
        }
    }

}
```

## 3. 스프링 빈 조회- 기본

- 가장 간단한 조회 방법:
- AnnotationConfigApplicationContext.getBean( Bean 이름, Type ) //이름과 타입으로 조회
- AnnotationConfigApplicationContext.getBean( Type ) //Type으로만 조회

- 조회 대상 Bean이 없으면 NoSuchBeanDefinitionException 발생

- test/hello.core/beanfind에 ApplicationContextBasicFindTest.class 생성

```java
package hello.core.beanfind;

import hello.core.AppConfig;
import hello.core.member.MemberService;
import hello.core.member.MemberServiceImpl;
import org.assertj.core.api.Assertions;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.NoSuchBeanDefinitionException;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class ApplicationContextBasicFindTest {

    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

    @Test
    @DisplayName("빈 이름으로 조회")
    void findBeanByName(){
        MemberService memberService = ac.getBean("memberService", MemberService.class);
        //memberService가 MemberServiceImpl의 Instance인지
        Assertions.assertThat(memberService).isInstanceOf(MemberServiceImpl.class);
    }

    @Test
    @DisplayName("이름 없이 타입으로만 조회")
    void findBeanByType(){
        MemberService memberService = ac.getBean(MemberService.class);
        //memberService가 MemberServiceImpl의 Instance인지
        Assertions.assertThat(memberService).isInstanceOf(MemberServiceImpl.class);
    }

    @Test
    @DisplayName("구체 타입으로 조회")
    void findBeanByName2(){
        //구체 타입으로도 조회가 가능하다. 하지만 이런 경우 유연성이 떨어진다.
        MemberServiceImpl memberService = ac.getBean("memberService", MemberServiceImpl.class);
        Assertions.assertThat(memberService).isInstanceOf(MemberServiceImpl.class);
    }

    @Test
    @DisplayName("빈 이름으로 조회 실패")
    void findBeanByNameX(){
        //ac.getBean("xxxxx", MemberService.class);

        //오른쪽 로직을 실행시켰을 때(ac.getBean("xxxxx", MemberService.class))
        //왼쪽 Exception(NoSuchBeanDefinitionException)이 터지면 테스트 성공
        org.junit.jupiter.api.Assertions.assertThrows(NoSuchBeanDefinitionException.class,
                () ->  ac.getBean("xxxxx", MemberService.class));
    }

}
```

## 4. 스프링 빈 조회 - 동일한 타입이 둘 이상일 때

- NoUniqueBeanDefinitionException 발생!

- test/hello.core/beanfind에 ApplicationContextSameBeanFindTest.class 생성

```java
package hello.core.beanfind;

import hello.core.AppConfig;
import hello.core.discount.DiscountPolicy;
import hello.core.member.MemberRepository;
import hello.core.member.MemoryMemberRepository;
import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.NoUniqueBeanDefinitionException;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.Map;

public class ApplicationContextSameBeanFindTest {

    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(SameBeanConfig.class);

    @Test
    @DisplayName("타입으로 조회시, 같은 타입이 둘 이상 있으면 중복 오류가 발생한다.")
    void findBeanByTypeDuplicate(){
        Assertions.assertThrows(NoUniqueBeanDefinitionException.class,
                () -> ac.getBean(MemberRepository.class));
    }

    @Test
    @DisplayName("타입으로 조회시 같은 타입이 둘 이상 있으면, Bean 이름을 지정하면 된다.")
    void findBeanByName(){
       MemberRepository memberRepository = ac.getBean("memberRepository1", MemberRepository.class);
        org.assertj.core.api.Assertions.assertThat(memberRepository).isInstanceOf(MemberRepository.class);
    }

    @Test
    @DisplayName("특정 타입을 모두 조회하기")
    void findAllBeanByType(){
        Map<String, MemberRepository> beansOfType = ac.getBeansOfType(MemberRepository.class);

        for (String key : beansOfType.keySet()) {
            System.out.println("key = " + key + " value = " + beansOfType.get(key));
        }

        System.out.println("beansOfType = " + beansOfType);
        org.assertj.core.api.Assertions.assertThat(beansOfType.size()).isEqualTo(2);
    }

    //Class 안에서 static : Scope가 현재 클래스
    //즉, 현재 Class 내부에서만 쓰겠다!
    @Configuration
    static class SameBeanConfig{
        //Test 내부에서 임시로 사용할 Config Class

        @Bean
        public MemberRepository memberRepository1(){
            //실제로도 , Parameter가 다르거나 한 경우, Type은 같지만 다른 Bean으로 등록될 수 있다.
            //
            return new MemoryMemberRepository();
        }

        @Bean
        public MemberRepository memberRepository2(){
            return new MemoryMemberRepository();
        }
    }
}
```

## 5. 스프링 빈 조회 - 상속 관계

- >> 부모 타입으로 조회하면, 자식 타입도 함께 조회한다. <<
- 그래서, 모든 자바 객체의 최고 부모인 'Object' 타입으로 조회하면, 모든 스프링 빈을 조회한다.

![Spring%20%E1%84%92%E1%85%A2%E1%86%A8%E1%84%89%E1%85%B5%E1%86%B7%20%E1%84%8B%E1%85%AF%E1%86%AB%E1%84%85%E1%85%B5%20%E1%84%89%E1%85%B3%E1%84%91%E1%85%B3%E1%84%85%E1%85%B5%E1%86%BC%20%E1%84%8F%E1%85%A5%E1%86%AB%E1%84%90%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%82%E1%85%A5%E1%84%8B%E1%85%AA%20%E1%84%89%E1%85%B3%E1%84%91%E1%85%B3%E1%84%85%E1%85%B5%E1%86%BC%20%E1%84%87%E1%85%B5%201622caf205e5420eab4a8ed0ec3b075f/Untitled%201.png](Spring%20%E1%84%92%E1%85%A2%E1%86%A8%E1%84%89%E1%85%B5%E1%86%B7%20%E1%84%8B%E1%85%AF%E1%86%AB%E1%84%85%E1%85%B5%20%E1%84%89%E1%85%B3%E1%84%91%E1%85%B3%E1%84%85%E1%85%B5%E1%86%BC%20%E1%84%8F%E1%85%A5%E1%86%AB%E1%84%90%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%82%E1%85%A5%E1%84%8B%E1%85%AA%20%E1%84%89%E1%85%B3%E1%84%91%E1%85%B3%E1%84%85%E1%85%B5%E1%86%BC%20%E1%84%87%E1%85%B5%201622caf205e5420eab4a8ed0ec3b075f/Untitled%201.png)

- test/hello.core/beanfind에 ApplicationContextExtendsFindTest.class 생성

```java
package hello.core.beanfind;

import hello.core.discount.DiscountPolicy;
import hello.core.discount.FixDiscountPolicy;
import hello.core.discount.RateDiscountPolicy;
import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.NoUniqueBeanDefinitionException;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.Map;

public class ApplicationContextExtendsFindTest {

    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(TestConfig.class);

    @Test
    @DisplayName("부모 타입으로 조회시, 자식이 둘 이상 있으면 중복 오류가 발생한다.")
    void findBeanByParentTypeDuplicate(){
        Assertions.assertThrows(NoUniqueBeanDefinitionException.class,
                ()-> ac.getBean(DiscountPolicy.class));
    }

    @Test
    @DisplayName("부모 타입으로 조회시, 자식이 둘 이상 있으면 빈 이름을 지정하면 된다.")
    void findBeanByParentTypeBeanName(){
        DiscountPolicy rateDiscountPolicy = ac.getBean("rateDiscountPolicy", DiscountPolicy.class);

        org.assertj.core.api.Assertions.assertThat(rateDiscountPolicy).isInstanceOf(RateDiscountPolicy.class);
    }

    @Test
    @DisplayName("특정 하위 타입으로 조회")
    void findBeanBySubType(){
        RateDiscountPolicy bean = ac.getBean(RateDiscountPolicy.class);
        org.assertj.core.api.Assertions.assertThat(bean).isInstanceOf(RateDiscountPolicy.class);

    }
    
    @Test
    @DisplayName("부모 타입으로 모두 조회하기")
    void findAllBeanByParentType(){
        Map<String, DiscountPolicy> beansOfType = ac.getBeansOfType(DiscountPolicy.class);
        org.assertj.core.api.Assertions.assertThat(beansOfType.size()).isEqualTo(2);

        //연습을 위해 Print 하긴 하지만, 실제로는 이렇게 하면 안 된다.
        //Test는 시스템이 확인하고 결정하도록 해야 한다. 사람이 눈으로 직접 보고 확인하게 해선 안 된다.
        for (String key : beansOfType.keySet()) {
            System.out.println("key = " + key + " value = " + beansOfType.get(key));
        }

    }

    //Spring에 있는 모든 Bean까지 다 나온다!
    @Test
    @DisplayName("부모 타입으로 모두 조회하기 - Object")
    void findAllBeanByObjectType(){
        Map<String, Object> beansOfType = ac.getBeansOfType(Object.class);

        for (String key : beansOfType.keySet()) {
            System.out.println("key = " + key + "value = " + beansOfType.get(key));
        }
    }

    @Configuration
    static class TestConfig{

        @Bean
        public DiscountPolicy rateDiscountPolicy(){
            return new RateDiscountPolicy();
        }

        @Bean
        public DiscountPolicy fixDiscountPolicy(){
            return new FixDiscountPolicy();
        }
    }

}
```

## 6. BeanFactory와 ApplicationContext

![Spring%20%E1%84%92%E1%85%A2%E1%86%A8%E1%84%89%E1%85%B5%E1%86%B7%20%E1%84%8B%E1%85%AF%E1%86%AB%E1%84%85%E1%85%B5%20%E1%84%89%E1%85%B3%E1%84%91%E1%85%B3%E1%84%85%E1%85%B5%E1%86%BC%20%E1%84%8F%E1%85%A5%E1%86%AB%E1%84%90%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%82%E1%85%A5%E1%84%8B%E1%85%AA%20%E1%84%89%E1%85%B3%E1%84%91%E1%85%B3%E1%84%85%E1%85%B5%E1%86%BC%20%E1%84%87%E1%85%B5%201622caf205e5420eab4a8ed0ec3b075f/Untitled%202.png](Spring%20%E1%84%92%E1%85%A2%E1%86%A8%E1%84%89%E1%85%B5%E1%86%B7%20%E1%84%8B%E1%85%AF%E1%86%AB%E1%84%85%E1%85%B5%20%E1%84%89%E1%85%B3%E1%84%91%E1%85%B3%E1%84%85%E1%85%B5%E1%86%BC%20%E1%84%8F%E1%85%A5%E1%86%AB%E1%84%90%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%82%E1%85%A5%E1%84%8B%E1%85%AA%20%E1%84%89%E1%85%B3%E1%84%91%E1%85%B3%E1%84%85%E1%85%B5%E1%86%BC%20%E1%84%87%E1%85%B5%201622caf205e5420eab4a8ed0ec3b075f/Untitled%202.png)

- BeanFactory
    - 스프링 컨테이너의 최상위 인터페이스
    - 스프링 빈을 관리/조회하는 기능을 담당한다.
    - getBean()을 제공한다.

- ApplicationContext
    - BeanFactory의 기능을 모두 상속받아서 제공한다.
    - Bean 관리 이외에도, 수많은 부가기능을 제공한다. (Application 제작시 공통적으로 필요한 기능들을 제공)
        1. 메시지소스를 활용한 국제화 기능
            - 한국에서 들어오면 한국어, 영어권에서 들어오면 영어로 출력
        2. 환경변수
            - 로컬, 개발, 운영등을 구분해서 처리
            - 로컬 : 내 컴퓨터에서 직접 개발할때
            - 개발 :  TEST 서버를 사용
            - 운영 : 실제 Proeduct로 나가는 운영 환경
        3. 애플리케이션 이벤트 
            - 이벤트를 발행하고 구독하는 모델을 편리하게 지원
        4. 편리한 리소스 조회
            - 파일, ClassPath, 외부 URL에서 리소스를 쓸 때 추상화 기능을 제공

## 7. 다양한 설정 형식 지원 - 자바 코드, XML

![Spring%20%E1%84%92%E1%85%A2%E1%86%A8%E1%84%89%E1%85%B5%E1%86%B7%20%E1%84%8B%E1%85%AF%E1%86%AB%E1%84%85%E1%85%B5%20%E1%84%89%E1%85%B3%E1%84%91%E1%85%B3%E1%84%85%E1%85%B5%E1%86%BC%20%E1%84%8F%E1%85%A5%E1%86%AB%E1%84%90%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%82%E1%85%A5%E1%84%8B%E1%85%AA%20%E1%84%89%E1%85%B3%E1%84%91%E1%85%B3%E1%84%85%E1%85%B5%E1%86%BC%20%E1%84%87%E1%85%B5%201622caf205e5420eab4a8ed0ec3b075f/Untitled%203.png](Spring%20%E1%84%92%E1%85%A2%E1%86%A8%E1%84%89%E1%85%B5%E1%86%B7%20%E1%84%8B%E1%85%AF%E1%86%AB%E1%84%85%E1%85%B5%20%E1%84%89%E1%85%B3%E1%84%91%E1%85%B3%E1%84%85%E1%85%B5%E1%86%BC%20%E1%84%8F%E1%85%A5%E1%86%AB%E1%84%90%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%82%E1%85%A5%E1%84%8B%E1%85%AA%20%E1%84%89%E1%85%B3%E1%84%91%E1%85%B3%E1%84%85%E1%85%B5%E1%86%BC%20%E1%84%87%E1%85%B5%201622caf205e5420eab4a8ed0ec3b075f/Untitled%203.png)

- Annotation 기반 자바 코드 설정 사용
    - 지금까지 사용했던 것
    - AnnotationConfigApplicationContext(AppConfig.class)'

- XML 기반 설정 사용
    - 최근에는 잘 사용하지 않지만, 많은 레거시 프로젝트들이 XML로 되어 있다.
    - XML을 사용하면 컴파일 없이 설정 정보를 변경할 수 있는 장점이 있으므로 알아는 두는 것이 좋다.
    - GenericXmlApplicationContext를 사용하면서 xml 설정 파일을 넘기면 된다.

- test/hello.core/xml package 생성 후 XmlAppContext.class 생성

```java
package hello.core.xml;

import hello.core.member.MemberService;
import org.assertj.core.api.Assertions;
import org.junit.jupiter.api.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.GenericXmlApplicationContext;

public class XmlAppContext {

    @Test
    void xmlAppContext(){
        ApplicationContext ac = new GenericXmlApplicationContext("appConfig.xml");
        MemberService memberService = ac.getBean("memberService", MemberService.class);
        Assertions.assertThat(memberService).isInstanceOf(MemberService.class);
    }
}
```

- main/resources/appConfig.xml 생성

    (리소스 파일은 resources 폴더 밑으로 간다.)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="memberService" class ="hello.core.member.MemberServiceImpl">

        <!-- 생성자를 넘겨준다. 하지만 이떄 memberRepository 가 없으므로 memberRepository 도 등록해줘야한다. -->
        <constructor-arg name="memberRepository" ref="memberRepository" />
    </bean>

    <bean id="memberRepository" class="hello.core.member.MemoryMemberRepository"/>

    <bean id="orderService" class="hello.core.order.OrderServiceImpl">
        <constructor-arg name="memberRepository" ref="memberRepository" />
        <constructor-arg name="discountPolicy" ref="discountPolicy" />
    </bean>

    <bean id="discountPolicy" class="hello.core.discount.RateDiscountPolicy"/>
</beans>
```

## 8. 스프링 빈 설정 메타 정보 - BeanDefinition

- 7) 과 같이 여러 설정 형식을 지원하기 위해서 "BeanDefinition" 이라는 추상화를 사용

- 역할과 구현을 개념적으로 나눈 것!
    - XML을 읽어 BeanDefinition을 만든다.
    - Java Code를 읽어 BeanDefinition을 만든다.
    - Spring Container는 Java Code, XML은 모른다. 오로지 BeanDefinition만 알면 된다.

- BeanDefinition을 빈 설정 메타정보라고 한다.
    - @Bean, <Bean>당 각각 하나씩 메타 정보가 생성된다.

- Spring Container는 이 메타정보를 기반으로 Spring Bean을 생성한다.

![Spring%20%E1%84%92%E1%85%A2%E1%86%A8%E1%84%89%E1%85%B5%E1%86%B7%20%E1%84%8B%E1%85%AF%E1%86%AB%E1%84%85%E1%85%B5%20%E1%84%89%E1%85%B3%E1%84%91%E1%85%B3%E1%84%85%E1%85%B5%E1%86%BC%20%E1%84%8F%E1%85%A5%E1%86%AB%E1%84%90%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%82%E1%85%A5%E1%84%8B%E1%85%AA%20%E1%84%89%E1%85%B3%E1%84%91%E1%85%B3%E1%84%85%E1%85%B5%E1%86%BC%20%E1%84%87%E1%85%B5%201622caf205e5420eab4a8ed0ec3b075f/Untitled%204.png](Spring%20%E1%84%92%E1%85%A2%E1%86%A8%E1%84%89%E1%85%B5%E1%86%B7%20%E1%84%8B%E1%85%AF%E1%86%AB%E1%84%85%E1%85%B5%20%E1%84%89%E1%85%B3%E1%84%91%E1%85%B3%E1%84%85%E1%85%B5%E1%86%BC%20%E1%84%8F%E1%85%A5%E1%86%AB%E1%84%90%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%82%E1%85%A5%E1%84%8B%E1%85%AA%20%E1%84%89%E1%85%B3%E1%84%91%E1%85%B3%E1%84%85%E1%85%B5%E1%86%BC%20%E1%84%87%E1%85%B5%201622caf205e5420eab4a8ed0ec3b075f/Untitled%204.png)

- AnnotationConfigApplicationContext는, AnnotationBeanDefinitionReader를 사용해, AppConfig.class를 읽고, BeanDefinition을 생성한다.
- 마찬가지로, GenericXmlApplicationContext는 XmlBeanDefinitionReader를 사용해, appConfig.xml을 읽고, BeanDefinition을 생성한다.

- 만약 새로운 설정 정보가 추가되면, xxxBeanDefinitionReader를 만들어 BeanDefinition을 생성하면 된다.

- BeanDefinition의 정보
    - BeanClassName :  생성할 Bean의 클래스명 ( 자바 설정처럼 팩토리 역할의 빈을 사용하면 없음)
    - factoryBeanName : 팩토리 역할의 빈을 사용할 경우 이름 (예 : appConfig)
    - factoryMethodName : 빈을 생성할 팩토리 메서드 지정. (예 : memberService)
    - Scope : 싱글톤 (기본값)
    - lazyInit : 스프링 컨테이너를 생성할 때 빈을 생성하는 것이 아니라, 실제 빈을 사용할때까지 최대한 생성을 지연처리하는지 여부
    - InitMethodName : 빈을 생성하고, 의존관계를 적용한 뒤 호출되는 초기화 메소드 명
    - DestroyMethodName : 빈의 생명주기가 끝나서 제거하기 직전에 호출되는 메서드 명
    - Constructor arguments, Properties : 의존관계 주입에서 사용 (자바 설정처럼 팩토리 역할의 빈을 사용하면 없음)

- Bean 설정을 확인할 수 있는 테스트 : /test/hello.core/beandefinition/BeanDefinitionTest.class

```java
package hello.core.beandefinition;

import hello.core.AppConfig;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.config.BeanDefinition;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class BeanDefinitionTest {

    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

    @Test
    @DisplayName("빈 설정 메타정보 확인")
    void findApplicationBean(){
        String[] beanDefinitionNames = ac.getBeanDefinitionNames();
        for (String beanDefinitionName : beanDefinitionNames) {
            BeanDefinition beanDefinition = ac.getBeanDefinition(beanDefinitionName);

            if(beanDefinition.getRole() == BeanDefinition.ROLE_APPLICATION){
                System.out.println("beanDefinitionName = " + beanDefinitionName +
                        " beanDefinition = " + beanDefinition);
            }

        }
    }
}
```

- BeanDefinition을 직접 생성해서 등록할 수도 있지만, 실무에서는 거의 사용할 일이 없다.
- > 스프링은 다양한 형태의 설정 정보를 BeanDefinition으로 추상화 해서 활용한다 <
- 가끔 스프링 코드나, 오픈 소스 코드를 볼 때 BeanDefinition이란 것이 보이면 위의 내용을 상기하면 된다.