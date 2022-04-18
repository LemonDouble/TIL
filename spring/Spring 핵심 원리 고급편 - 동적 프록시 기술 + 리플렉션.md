# Spring 핵심 원리 고급편 - 동적 프록시 기술 + 리플렉션

# 1. 리플렉션

- Proxy 패턴을 통해, 기존 코드 변경 없이 추가 기능을 추가할 수 있었다.
    - 하지만, 대상 클래스(혹은 인터페이스 수만큼) 새로운 클래스를 만들어야 한다!
    - 하지만, Proxy가 추가하는 소스코드는 거의 똑같은 모양을 하고 있다.

- Java가 제공하는 JDK 동적 프록시나, CGLIB 같은 기술을 사용하면 동적 프록시 생성 가능
    - Proxy 적용 코드를 하나만 만들고, 나머지는 동적으로 생성 가능하다.

### 리플렉션(Reflection)

- 클래스/메서드 메타정보를 동적으로 획득하고
- 코드도 동적으로 호출할 수 있는 기술

- 왜 리플렉션이 필요한가?

- 다음과 같은 상황에서
    - 앞뒤 로직은 같은데(공통) 중간의 메서드 하나가 다른 상황
    - 공통된 메서드로 뽑아낼 수 있을까?
- 이런 경우에 Reflection을 사용할 수 있음

```java
@Slf4j
public class ReflectionTest {

    @Test
    public void reflection0(){
        Hello target = new Hello();

        // 공통 로직 1 시작
        log.info("start");
        String result1 = target.callA(); // 로직은 같지만 호출하는 메서드가 다름
        log.info("result={}", result1);
        // 공통 로직 1 종료

        // 공통 로직 2 시작
        log.info("start");
        String result2 = target.callB(); // 로직은 같지만 호출하는 메서드가 다름
        log.info("result={}", result2);
        //공통 로직 2 종료
    }

    @Slf4j
    static class Hello{
        public String callA(){
            log.info("callA");
            return "A";
        }
        public String callB(){
            log.info("callB");
            return "B";
        }
    }
}
```

- 물론 Lambda 사용할 수도 있다. 하지만 Lambda 사용하기 어려운 경우라 가정하자.

### Reflection 사용

- 클래스 메타 정보를 획득하고
    - 메서드 메타 정보를 획득하고
    - 메서드 메타 정보로 실제 메서드를 호출할 수 있다.
- 이를 통해, Paramter로 메서드 정보를 보내거나, 동적으로 변경할 수 있다!

```java
@Test
    public void reflection1() throws ClassNotFoundException, NoSuchMethodException, InvocationTargetException, IllegalAccessException {
        // 클래스 메타 정보를 얻는다.
        Class classHello = Class.forName("hello.proxy.jdkdynamic.ReflectionTest$Hello");

        // target 생성
        Hello target = new Hello();
        // callA 메서드 정보 얻기 ( getMethod가 String으로 바뀌었으므로, Parameter로 넘기거나 할 수 있다.)
        Method methodCallA = classHello.getMethod("callA");
        // 동적으로 Method call
        Object result1 = methodCallA.invoke(target);
        log.info("result1={}", result1);

        //callB 메서드 정보 얻기
        Method methodCallB = classHello.getMethod("callB");
        Object result2 = methodCallB.invoke(target);
        log.info("result2={}", result2);
    }
```

- 따라서, 공통 부분을 추출할 수 있게 된다!

```java
@Test
public void reflection2() throws ClassNotFoundException, NoSuchMethodException, InvocationTargetException, IllegalAccessException {
    // 클래스 메타 정보를 얻는다.
    Class classHello = Class.forName("hello.proxy.jdkdynamic.ReflectionTest$Hello");

    // target 생성
    Hello target = new Hello();
    // callA 메서드 정보 얻기
    Method methodCallA = classHello.getMethod("callA");
    dynamicCall(methodCallA, target);

    //callB 메서드 정보 얻기
    Method methodCallB = classHello.getMethod("callB");
    dynamicCall(methodCallB, target);
}

// 공통 부분을 메소드로 추출할 수 있다!
private void dynamicCall(Method method, Object target) throws InvocationTargetException, IllegalAccessException {
    log.info("start");
    Object result = method.invoke(target);
    log.info("result={}", result);
}
```

### 주의:

- 단, Reflection 기능은 큰 단점이 있다.
    - 런타임에 동작하므로, 컴파일 시점에 오류를 잡을 수 없다.
    - 만약 method 명에 오타를 쳤을 경우, 런타임 시점에 (앱이 잘 돌아가다) 오류가 난다!

- 따라서, 일반적인 경우는 Reflection을 사용하면 안 된다.
    - Framework 개발, 혹은 매우 예외적인 상황에서 조심해서 사용해야 한다.

# 2. JDK 동적 프록시 - 소개

- Interface 기반으로 Proxy를 동적으로 생성하는 기술. 따라서 Interface가 필수이다.

# 3. JDK 동적 프록시 - 예제 코드

- InvocationHandler 인터페이스를 구현해야 함. 이때 제공되는 파라미터
    - Object proxy : 프록시 자신
    - Method method : 호출한 메서드
    - Object[] args : 메서드를 호출할 때 전달한 인수

```java
package java.lang.reflect;
public interface InvocationHandler {
		public Object invoke(Object proxy, Method method, Object[] args)
		throws Throwable;
}
```

- 다음과 같이, Interface를 구현할 수 있다.

```java
@Slf4j
public class TimeInvocationHandler implements InvocationHandler {

    private final Object target;

    public TimeInvocationHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        log.info("TimeProxy 실행");
        long startTime = System.currentTimeMillis();
        
        // Proxy가 호출하는 로직 실행
        Object result = method.invoke(target, args);
        
        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;
        log.info("TimeProxy 종료, resultTime = {}", resultTime);
        return result;
    }
}
```

- 그리고 다음과 같이 호출해서 사용할 수 있다.

```java
@Test
public void dynamicA(){
    AInterface target = new AImpl();
    TimeInvocationHandler handler = new TimeInvocationHandler(target);

    // 클래스 로더 정보, Interface, 핸들러 로직을 넣어 주면 된다.
    AInterface proxy = (AInterface) Proxy.newProxyInstance(AInterface.class.getClassLoader(),
            new Class[]{AInterface.class}, handler);

    // call 메소드가 실행됨.
    // TimeInvocationHandler의 로직이 실행되는데,
    // 중간에 target의 call 메소드를 호출하게 됨.
    proxy.call();
    log.info("targetClass={}", target.getClass());
    log.info("proxyClass={}", proxy.getClass());
}
```

### 실행 순서

1. 클라이언트가 JDK 동적 프록시의 call()을 실행
2. JDK 동적 프록시는 handler.invoke를 실행
3. 이때, TimeInvokationHandler가 Handler 구현체이므로, 오버라이딩된 invoke 메소드가 실행
4. TimeInvokationHandler는 시간 측정 로직 실행, 중간에 method.invoke(target,args) 실행해 실제 객체 (AImpl 혹은 BImpl) 을 호출
5. 해당 실제 객체의 call() 실행
6. 실제 객체의 로직이 완료되면, TimeInvokationHandler로 돌아오고 측정 시간 출력 후 반환

### 정리

- TimeInvokationHandler라는 공통 로직 하나로, 여러 클래스에 공통 사용 가능
- Proxy 클래스 100개씩 안 만들어도 된다!

# 3. JDK 동적 프록시 - 적용

- 다음과 같이 직접 Proxy 만들고

```java
public class LogTraceBasicHandler implements InvocationHandler {

    private final Object target;
    private final LogTrace logTrace;

    public LogTraceBasicHandler(Object target, LogTrace logTrace) {
        this.target = target;
        this.logTrace = logTrace;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        TraceStatus status = null;

        try{
            // OrderRepository.request() 처럼 나온다. 호출한 클래스.호출한 메소드명
            String message = method.getDeclaringClass().getSimpleName() + "." + method.getName() + "()";
            status =  logTrace.begin(message);
            // target 호출
            Object result = method.invoke(target, args);
            logTrace.end(status);
            return result;
        }catch(Exception e){
            logTrace.exception(status, e);
            throw e;
        }
    }
}
```

- 마찬가지로, Configuration에서 Bean 등록시 Proxy를 반환하도록 변경

```java
@Configuration
public class DynamicProxyBasicConfig {

    @Bean
    public OrderRepositoryV1 orderRepositoryV1(LogTrace logTrace){
        OrderRepositoryV1 orderRepository = new OrderRepositoryV1Impl();
        return (OrderRepositoryV1) Proxy.newProxyInstance(
                OrderRepositoryV1.class.getClassLoader(),
                new Class[]{OrderRepositoryV1.class},
                new LogTraceBasicHandler(orderRepository, logTrace)
        );
    }

    @Bean
    public OrderServiceV1 orderServiceV1(LogTrace logTrace){
        OrderServiceV1 orderService = new OrderServiceV1Impl(orderRepositoryV1(logTrace));
        return (OrderServiceV1) Proxy.newProxyInstance(
                OrderServiceV1.class.getClassLoader(),
                new Class[]{OrderServiceV1.class},
                new LogTraceBasicHandler(orderService, logTrace)
                );
    }

    @Bean
    public OrderControllerV1 orderControllerV1(LogTrace logTrace){
        OrderControllerV1 orderController = new OrderControllerV1Impl(orderServiceV1(logTrace));
        return (OrderControllerV1) Proxy.newProxyInstance(
                OrderServiceV1.class.getClassLoader(),
                new Class[]{OrderControllerV1.class},
                new LogTraceBasicHandler(orderController, logTrace)
        );
    }
}
```

- 하지만 이런 경우,  noLog() 함수도 호출되면 로그 남기는 문제 있다.
    - noLog()메서드는 보안적인 문제로,로그 남기면 안 된다고 가정하자.

# 4. JDK 동적 프록시 적용 - 2

- Pattern Match 통해서 Match 되는 경우에만 로그 남기도록 수정

- 외부에서 Pattern 주입받도록 핸들러 수정

```java
public class LogTraceFilterHandler implements InvocationHandler {

    private final Object target;
    private final LogTrace logTrace;
    private final String[] patterns;

    public LogTraceFilterHandler(Object target, LogTrace logTrace, String[] patterns) {
        this.target = target;
        this.logTrace = logTrace;
        this.patterns = patterns;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

        // 메서드 이름 필터
        // 특정 메서드 이름이 매칭되는 경우에만 LogTrace 로직을 실행 (WhiteList)
        String methodName = method.getName();
        // save, reqeust, reque*, *est (Pattern Matching)
        if(!PatternMatchUtils.simpleMatch(patterns, methodName)){
            // 만약 매칭 안 되면, 로그 찍으면 안 되니 바로 넘겨준다.
            return method.invoke(target, args);
        }

        TraceStatus status = null;

        try{
            // OrderRepository.request() 처럼 나온다. 호출한 클래스.호출한 메소드명
            String message = method.getDeclaringClass().getSimpleName() + "." + method.getName() + "()";
            status =  logTrace.begin(message);
            // target 호출
            Object result = method.invoke(target, args);
            logTrace.end(status);
            return result;
        }catch(Exception e){
            logTrace.exception(status, e);
            throw e;
        }
    }
}
```

- 외부에서 Pattern 주입 (Configuration)

```java
@Configuration
public class DynamicProxyFilterConfig {

    // 로그를 남길 Filter들
    private static final  String[] PATTERNS = {"request*", "order*", "save*"};
    @Bean
    public OrderRepositoryV1 orderRepositoryV1(LogTrace logTrace){
        OrderRepositoryV1 orderRepository = new OrderRepositoryV1Impl();
        return (OrderRepositoryV1) Proxy.newProxyInstance(
                OrderRepositoryV1.class.getClassLoader(),
                new Class[]{OrderRepositoryV1.class},
                new LogTraceFilterHandler(orderRepository, logTrace, PATTERNS)
        );
    }

    @Bean
    public OrderServiceV1 orderServiceV1(LogTrace logTrace){
        OrderServiceV1 orderService = new OrderServiceV1Impl(orderRepositoryV1(logTrace));
        return (OrderServiceV1) Proxy.newProxyInstance(
                OrderServiceV1.class.getClassLoader(),
                new Class[]{OrderServiceV1.class},
                new LogTraceFilterHandler(orderService, logTrace, PATTERNS)
                );
    }

    @Bean
    public OrderControllerV1 orderControllerV1(LogTrace logTrace){
        OrderControllerV1 orderController = new OrderControllerV1Impl(orderServiceV1(logTrace));
        return (OrderControllerV1) Proxy.newProxyInstance(
                OrderServiceV1.class.getClassLoader(),
                new Class[]{OrderControllerV1.class},
                new LogTraceFilterHandler(orderController, logTrace, PATTERNS)
        );
    }
}
```

### 한계

- JDK 동적 프록시는 Interface가 필수로 필요
    - 그렇다면 바로 구체 클래스를 쓰는 경우는? → 적용 불가능하다.
    - 따라서 이를 해결하기 위해 CGLIB이라는 라이브러리가 등장했다.

# 5. CGLIB - 소개

- CGLIB : Code Generator Library
    - 바이트코드를 조작해 동적으로 클래스 생성
    - 구체 클래스만 가지고 동적 프록시를 만들어 낼 수 있음.
    - Spring 내부에 포함되어 있어서 Spring에서는 그냥 사용 가능

- CGLIB을 직접 사용하는 경우는 거의 없다!
    - Spring ProxyFactory를 이해하기 위한 징검다리 정도로만 알아두자
    

# 6. CGLIB - 예제 코드

- 다음과 같이 MethodInterceptor을 implement 하는 클래스를 생성한다.
    - 주의 : MethodInterceptor가 org.springframework.cglib 인지 확인!

```java
@Slf4j
public class TimeMethodInterceptor implements MethodInterceptor {

    private final Object target;

    public TimeMethodInterceptor(Object target) {
        this.target = target;
    }

    @Override
    public Object intercept(Object o, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        log.info("TimeProxy 실행");
        long startTime = System.currentTimeMillis();

        // 잘은 모르는데 methodProxy 쓰면 좀 더 빠르다고 한다...
        Object result = methodProxy.invoke(target, args);

        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;
        log.info("TimeProxy 종료, resultTime = {}", resultTime);
        return result;
    }
}
```

- 이후 다음과 같이 호출해 사용할 수 있다.

```java
@Test
public void cglib(){
    ConcreteService target = new ConcreteService();
    Enhancer enhancer = new Enhancer();

    // 구체 클래스를 기반으로, 상속 받은 프록시를 만들어야 함.
    enhancer.setSuperclass(ConcreteService.class);
    // Proxy에 적용할 실행 로직을 할당
    enhancer.setCallback(new TimeMethodInterceptor(target));

    ConcreteService proxy = (ConcreteService) enhancer.create();

    // target Class는 원 Class
    log.info("targetClass={}", target.getClass());
    // proxy Class는 ConcreteService$$EnhancerByCGLIB$$25d6b0e3 (Spring에서 봤던 그거)
    log.info("proxyClass={}", proxy.getClass());

    proxy.call();

}
```

### CGLIB의 제약

1. 부모 클래스의 생성자 체크가 필요하다.
    - 자식 클래스를 동적으로 생성하므로, 기본 생성자가 필요하다!!
2. 클래스에서 final 키워드가 붙으면 상속이 불가능하다.
    - CGLIB에서는 Exception 발생
3. 메서드에 final 키워드가 붙으면 해당 메서드를 오버라이딩 할 수 없다.
    - CGLIB에서는 Proxy 로직이 동작하지 않는다.

# 7. 동적 프록시 기능 - 정리

- 여러 문제가 남아 있다.
    - Interface 있는 경우 JDK 동적 프록시를 적용하고, 그렇지 않은 경우 CGLib 적용하려면?
    - 두 기술을 함께 사용할 때, JDK의 InvocationHandler와 CGLIB의 MethodInterceptor를 중복으로 만들어서 관리해야할까?