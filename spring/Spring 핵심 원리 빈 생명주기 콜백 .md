# Spring 핵심 원리 : 빈 생명주기 콜백

분류: Spring
작성일시: 2021년 7월 19일 오후 1:52

## 1. 빈 생명주기 콜백 시작

- 크게 3가지 방법으로 Bean Lifecycle Callback을 지원한다.
    1. 인터페이스 (InitializingBean, DisposableBean)
    2. 설정 정보에 초기화 메서드, 종료 메서드 지정
    3. @PostConstruct, @PreDestroy Annotation

- DB 커넥션 풀 : DB 연결에 시간이 많이 들기 때문에, Application이 시작할 때 DB와 연결을 미리 해 둔다.
- Network 소켓 : 마찬가지로 어플리케이션 시작 시 Socket을 미리 열어 둠

- 위와 같이 어플리케이션 시작 시점에 필요한 연결을 미리 해 두고, 어플리케이션 종료 시점에 연결을 모두 종료하는 작업을 진행하렴녀 객체의 초기화/종료 작업이 필요함

- 어플리케이션 시작 시점에 connect()를 호출해서 네트워크를 연결하고, 어플리케이션 종료 시점에 disconnect()를 호출해 네트워크 연결을 종료하는 예제

- 테스트용 NetworkClient 객체 생성 (test/.../lifecycle/NetworkClient.class)

```java
```

- 이후 다음과 같이 Test를 작성하고 해 보면 (.../lifecycle/BeanLifeCycleTest.class)

```java
```

- 다음과 같이 오류가 발생한다.

```java
```

- 객체가 생성자를 통해 생성될 때는 URL이 없어 null로 Connect 하고 (오류 발생)

    이후 setUrl 통해 값을 넣어주고 나서야 내부 필드에 url이 set되므로 위와 같은 결과가 나온다.

- 스프링 빈은 다음과 같은 LifeCycle 가진다.
    1. 객체 생성
    2. 의존관계 주입

    의존관계 주입이 완료되어야 Class 내부의 모든 Field를 정상적으로 사용할 수 있다.

- 따라서, 의존관계 주입이 끝난 뒤 초기화 (위의 Connect() 같은 것) 작업을 실행할 수 있다.
- Spring은, 의존관계 주입이 완료되면 스프링 빈에게 Callback Method 를 통해 초기화 시점을 알려준다.
- 또한 스프링 컨테이너가 종료되기 직전에 소멸 Callback을 준다.

- Spring Bean의 Event Lifecycle
    1. Spring 컨테이너 생성
    2. Spring Bean 생성
    3. 의존관게 주입
    4. 초기화 Callback
    5. 사용
    6. 소멸 전 Callback
    7. Spring 종료

- 초기화 Callback : 빈이 생성되고, 빈의 의존관계 주입이 완료된 후 호출
- 소멸 전 Callback : 빈이 소멸되기 직전에 호출

---

- 참고 ) 객체의 생성과 초기화를 분리하자.
    - 생성자 : 필수 정보(파라미터) 를 받고, 메모리를 할당해 객체를 생성하는 책임
    - 초기화 : 이 값을 사용해 외부 커넥션을 연결하는 등, 무거운 동작을 수행
    - 객체 생성(생성자) 와 초기화 부분을 명확하게 나누는 것이 유지보수 관점에서 좋다.

 

## 2. 인퍼테이스 InitializingBean, DisposableBean

- Spring 초기에 나온 방법, 최근에는 거의 사용하지 않는다.

- NetworkClient.class를 다음과 같이 수정

```java
```

- 출력 결과 :

```java
```

- InitializingBean은 afterPropertiesSet() 으로 초기화를 지원한다.
- DisposableBean은 destroy()로 소멸을 지원한다.

- 초기화, 소멸 인터페이스의 단점
    - 이 인터페이스는 Spring 전용 인터페이스, 따라서 코드가 Spring 전용 인터페이스에 의존한다.
    - 초기화, 소멸 메소드의 이름을 변경할 수 없다.
    - 내가 코드를 고칠 수 없는 외부 라이브러리에 적용할 수 없다.

## 3. 빈 등록 초기화, 소멸 메소드 등록

- Bean 등록 시에 초기화/소멸 메소드를 직접 등록하는 방법

- BeanLifeCycleTest.class를 다음과 같이 변경

```java
```

- 설정 정보 사용 특징
    - 메소드 이름을 자유롭게 줄 수 있다.
    - 스프링 빈이 스프링 코드에 의존하지 않는다.
    - 코드가 아니라 설정 정보를 사용하므로, 코드를 고칠 수 없는 외부 라이브러리에도 초기화/종료 메소드를 적용할 수 있다.

- 종료 메서드 추론 (@Bean으로 등록할때만 발생)
    - 라이브러리는 대부분 close, shutdown이라는 이름의 종료 메서드를 사용한다.
    - @Bean의 destroyMethod의 기본 값은 (inferred) (추론) 으로 등록되어 있다.
    - 이 추론 기능은, close, shutdown 이라는 메서드를 자동으로 호출해 준다.
    - 따라서, 직접 Spring Bean으로 등록하면 (보통) 종료 메서드는 따로 적어주지 않아도 잘 동작한다.
    - 추론 기능을 사용하기 싫다면, destroyMethod="" 처럼 명시적으로 공백을 지정해야 한다.

## 4. 어노테이션 @PostConstruct, @PreDestory

- 주로 사용한다!

- NetworkClient에서 다음과 같이 수정

```java
```

- 해당 Annotation은 Java에서 공식적으로 지원

```java
```

- @PostConstruct, @PreDestory 어노테이션 특징
    - 최신 Spring에서 가장 권장하는 방식이다.
    - Annotation 하나만 붙이면 되므로 매우 편리하다.
    - 위와 같이 Java에서 지원하는 기능이다. JSR-250라는 자바 표준이므로, Spring이 아닌 다른 컨테이너에서도 동작한다.
    - 컴포넌트 스캔과 잘 어울린다.
    - 유일한 단점으론 외부 라이브러리에는 적용하지 못 한다는 점이다.

---

- 정리)
    - Annotation을 기본적으로 사용하자.
    - 하지만 코드를 고칠 수 없는 외부 라이브러리를 초기화/종료해야 하면 @Bean을 사용하자.
