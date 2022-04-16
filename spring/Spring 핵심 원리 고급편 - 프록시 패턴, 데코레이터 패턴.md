# Spring 핵심 원리 고급편 - 프록시 패턴, 데코레이터 패턴

# 1. 프로젝트 생성

- Git log 확인..

# 2. 예제 프로젝트 만들기 - v1, v2, v3

- V1 - 인터페이스+ 구현 클래스를 Spring Bean으로 수동 등록
- V2 - 인터페이스 없이 구현 클래스만, Spring Bean으로 수동 등록
- V3 - Component Scan으로 Spring Bean 자동 등록

- Spring MVC는 @Controller 혹은 @RequestMapping 애노테이션이 있어야 스프링 컨트롤러로 인식
    - 해당 어노테이션은 인터페이스에도 사용할 수 있다!

- @ResponseBody : HTTP 메세지 컨버터를 사용하여 응답.
    - 마찬가지로 인터페이스에 사용 가능하다!

```java
@Import(AppV1Config.class)
@SpringBootApplication(scanBasePackages = "hello.proxy.app") //주의
public class ProxyApplication {

	public static void main(String[] args) {
		SpringApplication.run(ProxyApplication.class, args);
	}

}
```

- @Import
    - 클래스를 스프링 빈으로 등록
    - 보통은 Configuration 등록할 때 사용
    - 하지만 일반 Bean 등록할 때도 사용한다.

# 3. 요구사항 추가

- 기존 로그 추적기를 적용하는데, 추가 요구사항이 생김
    1. 원본 코드는 전혀 수정하지 않고, 로그 추적기를 적용
    2. 특정 메서드는 로그를 출력하지 않는 기능
        - 보안상, 일부는 로그를 출력하면 안 된다.
    3. 다양한 케이스에 적용이 가능해야 한다.
        - v1 : 인터페이스가 있는 구체 클래스에 적용
        - v2 : 인터페이스가 없는 구체 클래스에 적용
        - v3 : 컴포넌트 스캔에 대상 기능 적용

# 4. 프록시, 프록시 패턴, 데코레이션 패턴

- Proxy?
    - Client → Server 요청을 할 때, 중간에 Proxy가 끼여있다면?
    - Client → Proxy → Server. Client는 Proxy를 통해 서버를 간접 호출한다.
    - 이 과정에서, Proxy가 부가 기능 등을 실행할 수도 있다!
        - 또는, Proxy가 다른 Proxy에게 뭔가 더 요청할수도 있다!

- 대체 가능
    - Proxy가 되려면, Client 입장에서
        - Server에게 요청한 것인지
        - Proxy에게 요청한 것인지
        - 조차 몰라야 한다!
    - 즉, Server와 Proxy는 같은 Interface를 사용해야 한다!
    - 또한, Client가 사용하는 Server 객체를 Proxy로 변경해도, 똑같이 동작해야 한다!

- 서버와 Proxy가 같은 Interface 사용하면, DI를 통해 서로 바꿀 수 있다!

### Proxy의 주요 기능

- 접근 제어
    - 권한에 따른 접근 차단
    - 캐싱
    - 지연 로딩
- 부가 기능 추가
    - 원래 서버가 제공하는 기능에 더해, Proxy의 부가 기능 수행
        - 예) 요청 값, 응답을 변형
        - 예) 실행 시간을 측정해 추가 로그를 남긴다.

### 프록시와 데코레이터

- 둘 다 프록시를 사용하는 방법이지만, 의도(intent)에 따라
    - **접근 제어**가 목적이면 “**Proxy** 패턴”
    - **새로운 기능 추가**가 목적이면 “**Decorator** 패턴”

- Tip : 둘 다 구조상으론 Proxy를 사용해서 구분 불가하다. 따라서 의도로 구분한다!
- Tip 2 : Proxy는 객체에서도, 서버에서도, 네트워크에서도 발견될 수 있다!

# 5. 프록시 패턴 - 예제 코드 1

- 코드 참조..

# 6. 프록시 패턴 - 예제 코드 2

- 클라이언트가 Proxy를 호출하면, Proxy는 최종적으로 실제 객체를 호출해줘야 함
    - Proxy 입장에서의 실제 객체를 “Traget”이라고 함

```java
@Slf4j
public class CacheProxy implements Subject{

    // 호출할 실제 객체
    private Subject target;
    // 캐시할 데이터
    private String cacheValue;

    public CacheProxy(Subject target) {
        this.target = target;
    }

    @Override
    public String operation() {
        log.info("프록시 호출됨");
        if(cacheValue == null){
            cacheValue = target.operation();
        }
        return cacheValue;
    }
}
```

- Client는 Proxy를 부르고, Proxy는 Target을 부를 수도, 안 부를 수도 있다! (접근 제어)

# 7. 데코레이터 패턴 - 예제 코드 1

- 코드 참조..

# 8. 데코레이터 패턴 - 예제 코드 2

- 결과값을 받아 꾸미는 Decorator를 하나 추가해 사용 가능했다!

```java
@Slf4j
public class MessageDecorator implements Component{

    private Component component;

    public MessageDecorator(Component component) {
        this.component = component;
    }

    @Override
    public String operation() {
        log.info("MessageDecorator 실행");

        String result = component.operation();
        String decoResult = "*****" + result + "*****";

        log.info("MessageDecorator 꾸미기 적용 전={}, 적용 후={}", result, decoResult);
        return decoResult;
    }
}
```

# 9. 데코레이터 패턴 - 예제 코드 3 (Decorator Chain)

- 새로운 Decorator를 추가해서 Chain처럼 사용할 수 있다.
    - 중요한 점은, Client는 코드 변경 없이, Chain이 일어나는지 모른다!

```java
RealComponent realComponent = new RealComponent();
MessageDecorator messageDecorator = new MessageDecorator(realComponent);
TimeDecorator timeDecorator = new TimeDecorator(messageDecorator);
DecoratorPatternClient client = new DecoratorPatternClient(timeDecorator);

client.execute();
```

# 10. 프록시 패턴과 데코레이터 패턴 정리

- 이 때, Decorator는 코드의 중복이 있다.

```java
public class TimeDecorator implements Component{
// 이 부분들!
private Component component;

public TimeDecorator(Component component) {
    this.component = component;
}
```

- 항상 꾸며줄 대상이 있어야 한다.
    - 따라서 호출 대상인 Component를 항상 가지고 있어야 한다.
    - 따라서, 이를 해결하기 위해 Decorator라는 추상 클래스를 만들 수도 있다.
    - 이를 통해, Decorator라는 추상 클래스를 통해 실제 컴포넌트와 Decorator를 구분할 수도 있다.

- Proxy Pattern vs Decorator Pattern
    - 중요한 것은 의도(intent)
        - Proxy : 접근 제어의 >> 의도 << 를 가지고 Proxy를 사용
        - Decorator : 객체에 추가 책임(기능) 을 추가하기 위해 Proxy를 사용