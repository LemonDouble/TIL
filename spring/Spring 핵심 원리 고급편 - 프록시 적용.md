# Spring 핵심 원리 고급편 - 프록시 적용

# 1. 인터페이스 기반 프록시 - 적용

- Controller, Service, Repository의 Proxy 생성
    - 아래 코드는 Service 경우

```java
@RequiredArgsConstructor
public class OrderRepositoryInterfaceProxy implements OrderRepositoryV1 {

    private final OrderRepositoryV1 target;
    private final LogTrace logTrace;

    @Override
    public void save(String itemId) {
        TraceStatus status = null;

        try{
            status =  logTrace.begin("OrderRepository.request()");
            // target 호출
            target.save(itemId);
            logTrace.end(status);
        }catch(Exception e){
            logTrace.exception(status, e);
            throw e;
        }
    }
}
```

- 이후 Spring Bean을 등록하는데, 실제 객체가 Proxy를 통해 호출되도록 설정
- `// Client -> OrderControllerProxy -> Controller(진짜) -> orderServiceProxy -> Service(진짜) -> OrderRepositoryProxy -> Repository(진짜)`

```java
@Configuration
public class InterfaceProxyConfig {

    // 의존관계 설정
    // Client -> OrderControllerProxy -> Controller(진짜) ->
    // orderServiceProxy -> Service(진짜) -> OrderRepositoryProxy -> Repository(진짜)

    @Bean
    public OrderControllerV1 orderController(LogTrace logTrace){
        OrderControllerV1Impl controllerImpl = new OrderControllerV1Impl(orderService(logTrace));
        // Proxy를 반환, Proxy는 target과 연결되어 있음.
        // 따라서 Request -> Proxy -> Target(이 경우 실제 orderController)로 간다.
        return new OrderControllerInterfaceProxy(controllerImpl, logTrace);
    }

    @Bean
    public OrderServiceV1 orderService(LogTrace logTrace){
        OrderServiceV1Impl serviceImpl = new OrderServiceV1Impl(orderRepository(logTrace));
        return new OrderServiceInterfaceProxy(serviceImpl, logTrace);
    }

    @Bean
    public OrderRepositoryV1 orderRepository(LogTrace logTrace){
        OrderRepositoryV1Impl orderRepositoryImpl = new OrderRepositoryV1Impl();
        return new OrderRepositoryInterfaceProxy(orderRepositoryImpl, logTrace);
    }

		@Bean
			public LogTrace logTrace(){
				return new ThreadLocalLogTrace();
		}
}
```

- Proxy 패턴을 통해, Controller, Service, Repository는 코드를 전혀 수정하지 않고 로그 추적기가 도입되었다!

# 2. 구체 클래스 기반 프록시 - 예제 1

# 3. 구체 클래스 기반 프록시 - 예제 2

- 핵심 아이디어 : 부모 클래스 자리에는 자식 클래스가 들어갈 수 있다!

- 원래 Class

```java
@Slf4j
public class ConcreteLogic {

    public String operation(){
        log.info("Concrete logic 실행");
        return "data";
    }
}
```

- 이를 상속받는 Class를 구현한다. (Proxy)

```java
@Slf4j
public class TimeProxy extends ConcreteLogic{

    private ConcreteLogic realLogic;

    public TimeProxy(ConcreteLogic realLogic) {
        this.realLogic = realLogic;
    }

    @Override
    public String operation() {
        log.info("TimeDecorator 실행");
        long startTime = System.currentTimeMillis();

        String result = realLogic.operation();

        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;
        log.info("TimeDecorator 종료 resultTime={}", resultTime);
        return result;
    }
}
```

- 다음, Proxy를 사용해 호출한다!

```java
@Test
public void addProxy(){
    ConcreteLogic concreteLogic = new ConcreteLogic();
    TimeProxy timeProxy = new TimeProxy(concreteLogic);

    // 자식 객체도 할당 가능하다!
    ConcreteClient client = new ConcreteClient(timeProxy);

    client.execute();
}
```

# 4. 구체 클래스 기반 프록시 - 적용

- 다음과 같이 상속받는 클래스로 Proxy 만든다.
    - 이때, 부모 Class에 파라미터 받는 생성자 있다면, super() 를 불러줘야 한다.
    - 부모 클래스의 기능은 전혀 사용하지 않으므로, null 넣어줘도 상관 없다.

```java
public class OrderServiceConcreteProxy extends OrderServiceV2 {

    private final OrderServiceV2 target;
    private final LogTrace logTrace;

    public OrderServiceConcreteProxy(OrderServiceV2 target, LogTrace logTrace) {
        // Java 문법상 부모 클래스의 생성자를 반드시 만들어줘야 하므로, super() 붙는다.
        // 하지만, 부모 클래스의 기능은 하나도 쓰지 않기 떄문에 그냥 null 제공한다.
        // Interface 기반이면 이런 고민을 하지 않아도 된다.
        super(null);
        this.target = target;
        this.logTrace = logTrace;
    }

    @Override
    public void orderItem(String itemId) {
        TraceStatus status = null;
        try{
            status =  logTrace.begin("OrderService.request()");
            // target 호출
            target.orderItem(itemId);
            logTrace.end(status);
        }catch(Exception e){
            logTrace.exception(status, e);
            throw e;
        }
    }
}
```

- 이후 Spring Bean으로 등록할 때, 다음과 같이 실제 객체 대신 Proxy로 감싼 객체를 반환한다.

```java
@Configuration
public class ConcreteProxyConfig {

    @Bean
    public OrderRepositoryV2 orderRepositoryV2(LogTrace logTrace){
        OrderRepositoryV2 repositoryImpl = new OrderRepositoryV2();
        return new OrderRepositoryConcreteProxy(repositoryImpl, logTrace);
    }

    @Bean
    public OrderServiceV2 orderServiceV2(LogTrace logTrace){
        OrderServiceV2 serviceImpl = new OrderServiceV2(orderRepositoryV2(logTrace));
        return new OrderServiceConcreteProxy(serviceImpl, logTrace);
    }

    @Bean
    public OrderControllerV2 orderControllerV2(LogTrace logTrace){
        OrderControllerV2 controllerImpl = new OrderControllerV2(orderServiceV2(logTrace));
        return new OrderControllerConcreteProxy(controllerImpl, logTrace);
    }
}
```

# 5. 인터페이스 기반 프록시, 클래스 기반 프록시

- 상관없이, 기존 코드 변경 없이 LogTrace 기능을 추가할 수 있었다.

- Interface 기반 프록시에 비해 클래스 기반 프록시의 단점
    1. 클래스 기반 프록시는 해당 클래스에만 적용 가능 (Interface 기반 프록시는 **같은 Interface 사용하는 모든 클래스** 적용 가능)
    2. 상속을 사용하므로, 제한이 있다.
        - 부모 클래스의 생성자를 강제로 호출해야 한다. 4) 에서 한것처럼, super() 호출이 필수이다.
        - 클래스에 final이 붙으면 상속이 불가능
        - 메소드에 final이 붙으면 해당 메서드는 오버라이딩 불가능

- 하지만 Interface 기반 키워드는, Interface를 직접 구현해야 한다는 것이 불편하다.
    - 바뀔 일이 절대 없는 클래스는, 굳이 Interface를 만들 필요가 있을까?

- 클래스의 특성을 먼저 생각하고, 적절하게 섞어 사용하자.

- 하지만 문제가 있다.
    - LogTrace의 경우 결국 코드 자체는 똑같다.
    - 하지만 Class, Interface가 달라서 프록시를 수십개 만들었다.
    - 이를 해결하기 위해 동적 프록시 기술이란 게 있다.