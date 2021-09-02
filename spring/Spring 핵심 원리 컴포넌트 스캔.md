# Spring 핵심 원리 : 컴포넌트 스캔

분류: Spring
작성일시: 2021년 7월 16일 오후 10:25

## 1. 컴포넌트 스캔과 의존관계 자동 주입 시작하기

- 지금까지 Spring Bean을 등록할 때는, Java Code에서 @Bean이나, XML을 통해 직접 등록했다.
- 하지만 이렇게 등록해야 할 Spring Bean의 양이 많아지면 일일히 등록하기 힘들고, 설정 정보가 커지고 누락되는 문제도 발생한다.
- 따라서, 설정 정보가 없어도 자동으로 Spring Bean을 등록해 주는 컴포넌트 스캔이라는 기술을 제공한다.
- 또한, 의존관계도 자동으로 주입하는 @Autowired라는 기능도 제공한다.

- 새로운 AutoAppConfig.class를 만든다. (Appconfig랑 같은 디렉토리)

```java
```

- 컴포넌트 스캔은 이름 그대로, @Component 어노테이션이 붙은 클래스를 스캔해 Spring Bean으로 등록한다.
- 따라서, 각 클래스가 컴포넌트 스캔의 대상이 되도록 @Component 애노테이션을 붙여준다.
- MemoryMemberRepository, RateDiscountPolicy, MemberServiceImpl, OrderServiceImpl에 @Component 어노테이션을 붙여준다.
- MemberServiceImpl은 MemberRepository, OrderServiceImpl은 MemberRepository, DiscountPolicy가 필요하므로 생성자에 @Autowired 어노테이션을 붙여 준다.

![Spring%20%E1%84%92%E1%85%A2%E1%86%A8%E1%84%89%E1%85%B5%E1%86%B7%20%E1%84%8B%E1%85%AF%E1%86%AB%E1%84%85%E1%85%B5%20%E1%84%8B%E1%85%B4%E1%84%8C%E1%85%A9%E1%86%AB%E1%84%80%E1%85%AA%E1%86%AB%E1%84%80%E1%85%A8%20%E1%84%8C%E1%85%A1%E1%84%83%E1%85%A9%E1%86%BC%20%E1%84%8C%E1%85%AE%E1%84%8B%E1%85%B5%E1%86%B8%2056b71483deb04d889e1497be5227a047/Untitled.png](https://github.com/LemonDouble/TIL/blob/main/spring/img/Untitled%2017.png)

## 2. 탐색 위치와 기본 스캔 대상

- 모든 Java Class를 전부 스캔하면 시간이 오래 걸린다.
- 따라서 옵션으로, 필요한 위치부터 탐색하도록 시작할 수 있다.

```java
```

- 설정하지 않을 시, 기본값으론 @ComponentScan이 붙은 설정 정보 클래스의 패키지가 시작 위치가 된다.

- 권장하는 방법
    - 별도로 스캔 위치를 지정하지 않고, 설정 정보 클래스의 위치를 프로젝트 최상단에 둠
    - Spring Boot도 기본적으로 제공한다. (CoreApplication의 @SpringBootApplication 어노테이션이 @ComponentScan 어노테이션을 포함한다)
    - 관례적으로도 루트 디렉토리에 설정 클래스를 둔다.
    - 프로젝트 설정 정보는 프로젝트를 대표하는 정보이므로, 루트 위치에 두는 것이 좋기도 하다.

- 컴포넌트 스캔 기본 대상
    - @Component : 컴포넌트 스캔에서 사용
    - @Controller : 스프링 MVC 컨트롤러에서 사용
    - @Service : 스프링 비즈니스 로직에서 사용
    - @Repository : 스프링 데이터 접근 계층에서 사용
    - @Configuration : 스프링 설정 정보에서 사용

참고 ) Annotation에는 상속 관계는 없다. 어노테이션이 다른 어노테이션을 포함하는 것은 Java가 지원하는 기능이 아닌, Spring이 지원하는 기능이다.

- 컴포넌트 스캔의 용도 뿐 아니라, 다음 애노테이션에 따라 스프링이 부가 기능을 수행한다.
    - @Controller : 스프링 MVC 컨트롤러로 인식
    - @Repository : 스프링 데이터 접근 계층으로 인식하고, 데이터 계층 예외를 스프링 예외로 변환하는 기능을 제공한다.
    - @Configuration : 스프링 설정 정보로 인식하고, 안에 설정된 Bean들이 싱글톤을 유지하도록 추가 처리를 한다.
    - @Service : 특별한 처리를 하지 않는다. 대신 개발자들이 핵심 비즈니스 로직이 여기 있겠구나 하고 비즈니스 계층을 인식하는데 도움이 된다.

## 3. 필터

- includeFilters : 컴포넌트 스캔 대상을 추가로 지정한다.
- excludeFilters : 컴포넌트 스캔에서 제외할 대상을 지정한다.

- test/../scan/filter package를 만들고, 아래 코드들을 추가한다.

1. MyIncludeComponent.Annotation (Class 만드는 창에서 Annotation 선택)

```java
```

1. MyExcludeComponent.Annotation

```java
```

1. BeanA.class

```java
```

1. BeanB.class

```java
```

5.ComponentFilterAppConfigTest.class

```java
```

- MyIncludeComponent 는 스캔되고, MyExcludeComponent는 스캔되지 않는다.

- FilterType은 5가지 옵션이 있다.
    1. ANNOTATION : 기본값, Annotation을 인식해 동작한다.
        - org.example.SomeAnnotation
    2. ASSIGNABLE_TYPE : 지정한 타입과 자식 타입을 인식해 동작한다.
        - org.example.SomeClass
    3. ASPECTJ : ASPECTJ패턴 사용
        - org.example..*Service+
    4. REGEX : 정규 표현식
        - org\.example\.Default.*
    5. CUSTOM : TypeFilter라는 인터페이스를 직접 구현해 처리한다.
        - org.example.MyTypeFilter

- 예를 들어 BeanA도 빼려면 다음과 같이 추가하면 된다.

```java
```

- 하지만 @Component면 충분하기 때문에, IncludeFilters는 거의 사용하지 않는다.
- ExcludeFilters도, 자주 사용되진 않는다.

- 직접 필터를 만드는것보다, 스프링 기본 설정에 최대한 맞춰 사용하는 것이 권장되는 편이다.

## 4. 중복 등록과 충돌

- 컴포넌트 스캔에서 같은 빈 이름을 등록하는 경우에 대해 다룬다.
    1. 자동 빈 등록 & 자동 빈 등록 → ConflictingBeanDefinitionException 오류 발생!
    2. 자동 빈 등록 & 수동 빈 등록 → 수동 빈 등록이 우선권을 가진다.

- 자동 빈 등록 vs 수동 빈 등록
    - AutoAppConfig.class를 다음과 같이 수정

    ```java
    
    ```

    - 이후 test/.../scan/AutoAppConfigTest.class의 test 실행

        테스트 실행시, 로그에 자동 빈을 수동 빈으로 오버라이딩 했음이 남는다

        ```
        Overriding bean definition for bean 'memoryMemberRepository' with a different
        definition: replacing
        ```

- 의도적으로 오버라이딩을 했다면 괜찮지만, 보통은 설정이 꼬여서 이런 결과가 나오는 경우가 대부분이다
- 그래서 최근 스프링 부트에서는 수동/자동 빈 등록이 충돌하면 오류가 발생하도록 기본 값이 바뀌었다.
- CoreApplication 실행시 오류 발생하는 것을 확인할 수 있다!

    (bean 이름을 바꾸거나, overriding 설정을 켜라는 로그)

```
The bean 'memoryMemberRepository', defined in class path resource [hello/core/AutoAppConfig.class], could not be registered. A bean with that name has already been defined in file [C:\Spring\core\out\production\classes\hello\core\member\MemoryMemberRepository.class] and overriding is disabled.

Action:

Consider renaming one of the beans or enabling overriding by setting spring.main.allow-bean-definition-overriding=true
```
