# Spring 핵심 원리 고급편 - 스프링이 지원하는 프록시

# 1. Proxy Factory - 소개

- 동적 프록시를 사용할 때의 문제점
    - Interface 있는 경우 JDK 동적 Proxy 사용, 그렇지 않은 경우는 어떻게 CGLIB을 적용할까?
    - 두 기술을 함께 사용할 때, JDK 동적 프록시의 InvocationHander와 CGLIB의 MethodInterceptor를 중복으로 만들어서 관리해야 하나?
    - 특정 조건에 맞을 때만 Proxy 로직을 제공하는 기능도 공통으로 제공할 수 있을까? (noLog 메서드는 Logging 기능 제공하지 않으려면?)

### Spring이 해결한 방법

1. Interface 있는 경우 JDK 동적 Proxy 사용, 그렇지 않은 경우는 어떻게 CGLIB을 적용할까?
    - Interface를 통해 JDK 동적 프록시와 CGLIB을 추상화
    - 따라서 Proxy Factory만 구현하면, Spring이 JDK 혹은 CGLIB을 선택하여 프록시를 만들어 준다
2. 두 기술을 함께 사용할 때, JDK 동적 프록시의 InvocationHander와 CGLIB의 MethodInterceptor를 중복으로 만들어서 관리해야 하나?
    - Spring은 Advice라른 개념을 도입
    - 따라서 개발자는 Spring의 Advice만 구현하면 된다!
    - 결과적으로, JDK의 InvocationHandler, MethodInterceptor는 Advice를 호출!
3. 특정 조건에 맞을 때만 Proxy 로직을 제공하는 기능도 공통으로 제공할 수 있을까? (noLog 메서드는 Logging 기능 제공하지 않으려면?)
    - Spring은 Pointcut이라는 개념을 도입
    - Pointcut을 통해 Pattern Matching 제공!

- 개발자 입장에서의 요약 : Proxy 생성은 ProyFactory, 로직은 Advice로

# 2. 프록시 팩토리 - 예제 코드

- 다음과 같이 Advice를 구현
    - MethodInterceptor는 Interceptor를 상속하고, Interceptor는 Advice를 상속한다.
        - 따라서 MethodInterceptor를 Advice로 쓸 수 있다..
    - 패키지 명은 org.aopalliance.intercept.MethodInterceptor이다. (주의..)

```java
@Slf4j
public class TimeAdvice implements MethodInterceptor {

    @Override
    public Object invoke(MethodInvocation invocation) throws Throwable {
        log.info("TimeProxy 실행");
        long startTime = System.currentTimeMillis();

        // Proxy가 호출하는 로직 실행
        // Object result = method.invoke(target, args);
        // 위와 같이 할 필요 없이, invocation.proceed() 실행하면 된다.
        // 알아서 Target을 을 찾아 실행시키고, 결과를 받는다.
        // Target 정보는 invocation 안에 들어 있다. ( Proxy factory 생성할 때 target 정보를 미리 넘기기 때문에 가능)
        Object result = invocation.proceed();

        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;
        log.info("TimeProxy 종료, resultTime = {}", resultTime);
        return result;
    }
}
```

- 다음과 같이 사용할 수 있다.

```java
@Test
@DisplayName("인터페이스가 있으면 JDK 동적 프록시 실행")
void interfaceProxy(){
    ServiceInterface target = new ServiceImpl();
    ProxyFactory proxyFactory = new ProxyFactory(target);
    proxyFactory.addAdvice(new TimeAdvice());
    ServiceInterface proxy = (ServiceInterface) proxyFactory.getProxy();

    log.info("targetClass={}", target.getClass());
    //com.sun.proxy.$Proxy13 : JDK 동적 Proxy
    log.info("proxyClass={}", proxy.getClass());

    proxy.save();

    // ProxyFactory를 통해 만들었다면, 아래와 같이 proxy인지 확인할 수 있다.
    log.info("AopUtils.isAopProxy(proxy)={}", AopUtils.isAopProxy(proxy));
    // JDK Dynamic Proxy인지
    log.info("AopUtils.isJdkDynamicProxy(proxy)={}", AopUtils.isJdkDynamicProxy(proxy));
    // CGLIB Proxy인지도 알려 준다.
    log.info("AopUtils.isJdkDynamicProxy(proxy)={}", AopUtils.isCglibProxy(proxy));
}
```

# 3. 프록시 팩토리 - 예제 코드 2

- 같은 코드로 구체 Class만 있는 경우, CGLIB 사용하는 것을 볼 수 있음.

- 다음과 같이 proxyFactory.proxyTargetClass(true)를 주면
    - Interface가 있어도 CGLIB을 사용한다.

```java
@Test
@DisplayName("ProxyTargetClass 옵션을 사용하면, Interface가 있어도 CGLIB을 사용하고, Class 기반 프록시 사용")
void proxyTargetClass(){
    ServiceInterface target = new ServiceImpl();
    ProxyFactory proxyFactory = new ProxyFactory(target);
    // 항상 CGLIB 기반으로 만들도록 설정
    proxyFactory.setProxyTargetClass(true);
    proxyFactory.addAdvice(new TimeAdvice());
    ServiceInterface proxy = (ServiceInterface) proxyFactory.getProxy();

    log.info("targetClass={}", target.getClass());
    //ServiceImpl$$EnhancerBySpringCGLIB$$425cd04e : CGLIB Proxy
    log.info("proxyClass={}", proxy.getClass());

    proxy.save();

    // ProxyFactory를 통해 만들었다면, 아래와 같이 proxy인지 확인할 수 있다.
    log.info("AopUtils.isAopProxy(proxy)={}", AopUtils.isAopProxy(proxy));
    // JDK Dynamic Proxy인지
    log.info("AopUtils.isJdkDynamicProxy(proxy)={}", AopUtils.isJdkDynamicProxy(proxy));
    // CGLIB Proxy인지도 알려 준다.
    log.info("AopUtils.isCglibProxy(proxy)={}", AopUtils.isCglibProxy(proxy));
}
```

- TIP
    - Spring Boot는 AOP를 적용할 때, 기본적으로 proxyTargetClass = true로 사용한다.
    - 따라서, Interface가 있어도 항상 CGLIB을 사용하여 구체 클래스 기반으로 프록시를 생성한다.

# 4. 포인트컷, 어드바이스, 어드바이저

- **포인트컷(PointCut)**
    - 필터링 로직
    - 어디에 부가 기능을 적용할지? 어디에 적용하지 않을지?
    - 보통 Class, Method 이름으로 필터링

- **어드바이스 (Advice)**
    - Proxy가 호출하는 부가 기능
    - 프록시 로직!

- **어드바이저 (Advisor)**
    - 1개의 포인트컷 + 1개의 Advice
    - 포인트컷과 어드바이스를 둘 다 가지고 있어, 어디에 어떤 로직을 적용할지 아는 친구

- 위와 같이, 역할과 책임을 분리했다.
    - Pointcut : 대상 여부 확인하는 Filter 역할만 담당
    - Advice : 부가 기능 로직만 담당
    - Advisor : 둘을 합쳐 저장

# 5. 어드바이저 - 예제 코드 1

- Advisor는 다음과 같이 사용 가능

```java
@Test
public void advisorTest1(){
    ServiceInterface target = new ServiceImpl();
    ProxyFactory proxyFactory = new ProxyFactory(target);
    // Pointcut.TRUE : 항상 참인 Pointcut
    // TIP : proxyFactory.addAdvice(new TimeAdvice()) 는, 내부적으로 아래와 똑같이 동작한다 (True Pointcut으로 들어감)
    DefaultPointcutAdvisor advisor = new DefaultPointcutAdvisor(Pointcut.TRUE, new TimeAdvice());
    // add로 Advisor를 추가할 수 있다.
    proxyFactory.addAdvisor(advisor);

    ServiceInterface proxy = (ServiceInterface) proxyFactory.getProxy();

    proxy.save();
    proxy.find();
}
```

- Tip : addAdvice()는 내부적으로,항상 True를 반환하는 Pointcut이 들어간 Advisor로 변환되어 등록된다.

# 6. 어드바이저 - 예제 코드 2

- 기므로 코드 직접 참조 (직접 만들 일도 거의 없으므로..)
- ClassMatcher, MethodMatcher을 직접 구현하면, 호출되었을 때 Class와 Method가 Match되는지 확인해, Match되는 경우 Proxy를 적용한다.

# 7. Spring이 제공하는 포인트컷 - 예제 코드 3

- 다음과 같이 Spring이 제공하는 Pointcut을 편하게 사용 가능하다.

```java
// Spring이 제공하는 Pointcut
NameMatchMethodPointcut pointcut = new NameMatchMethodPointcut();
// Save라는 Method 명을 Match!
pointcut.setMappedName("save");
// 위 Pointcut을 추가
DefaultPointcutAdvisor advisor = new DefaultPointcutAdvisor(pointcut, new TimeAdvice());
```

- 대표적인 몇 포인트컷
    - NameMatchMethodPointcut : 메서드 이름 기반 매칭
        - 내부에서는 PatternMatchUtils 를 사용한다.
    - JdkRegexpMethodPointcut : JDK 정규 표현식 기반으로 포인트컷 매칭
    - TruePointcut : 항상 True 반환
    - AnnotationMatchingPointcut : Annotation으로 매칭
    
    - AspectJExpressionPointcut : AspectJ 표현식으로 매칭 (중요! 제일 많이 사용한다.)

# 8. 여러 어드바이저 함께 적용 - 예제 코드 4

1. 간단하게 적용하는 방법

```java
@Test
@DisplayName("여러 프록시")
public void multiAdvisorTest1(){
    // client -> proxy2(advisor2) -> proxy1(advisor1) -> target

    // 프록시 1 생성 (proxy1 -> target)
    ServiceInterface target = new ServiceImpl();
    ProxyFactory proxyFactory1 = new ProxyFactory(target);
    DefaultPointcutAdvisor advisor = new DefaultPointcutAdvisor(Pointcut.TRUE, new Advice1());
    proxyFactory1.addAdvisor(advisor);
    ServiceInterface proxy1 = (ServiceInterface) proxyFactory1.getProxy();

    // 프록시 2 생성 (proxy2 -> proxy1. 이래야 proxy2 -> proxy1 -> target 됨)
    ProxyFactory proxyFactory2 = new ProxyFactory(proxy1);
    DefaultPointcutAdvisor advisor2 = new DefaultPointcutAdvisor(Pointcut.TRUE, new Advice2());
    proxyFactory2.addAdvisor(advisor2);
    ServiceInterface proxy2 = (ServiceInterface) proxyFactory2.getProxy();

    // 실행
    proxy2.save();
}
```

- 하지만 문제점 있다!
    - Advisor를 추가할 때마다, Proxy를 새로 생성해야 한다.
    - Advisor가 100개라면, Proxy를 100개 만들어야 한다!
    - 따라서, 한 Proxy Factory에 여러 Advisor를 적용 가능하면 좋을 것 같다!

- 따라서 다음과 같이, 한 Proxy에 여러 Advisor을 호출할 수 있다.
    - 이때, 호출 순서는 “등록 순서” 와 같다.

```java
@Test
@DisplayName("하나의 프록시에 여러 어드바이저 적용")
public void multiAdvisorTest2(){
    // client -> proxy -> (advisor2) -> (advisor1) -> target

    DefaultPointcutAdvisor advisor1 = new DefaultPointcutAdvisor(Pointcut.TRUE, new Advice1());
    DefaultPointcutAdvisor advisor2 = new DefaultPointcutAdvisor(Pointcut.TRUE, new Advice2());

    // 프록시 1 생성 (proxy1 -> target)
    ServiceInterface target = new ServiceImpl();
    ProxyFactory proxyFactory = new ProxyFactory(target);

    // 2번 먼저 호출됨
    proxyFactory.addAdvisor(advisor2);
    // 그 다음 1번 호출 (순서대로)
    proxyFactory.addAdvisor(advisor1);

    ServiceInterface proxy = (ServiceInterface) proxyFactory.getProxy();

    // 실행
    proxy.save();
}
```

### 중요!!

- 이후, Spring AOP를 사용하면, AOP만큼 Proxy가 생성된다고 착각하는 경우 많다.
    - 실제로는, 최적화 진행해 Proxy는 하나로 만들고, 하나의 Proxy에 여러 Advisor를 적용한다.
    - 따라서 하나의 Target에 여러 AOP 적용되어도, Target마다 하나의 Proxy만 생성된다!

# 9. 프록시 팩토리 - 적용 1, 2

- 다음과 같이 Advice 정의한 후

```java
public class LogTraceAdvice implements MethodInterceptor {

    private final LogTrace logTrace;

    public LogTraceAdvice(LogTrace logTrace) {
        this.logTrace = logTrace;
    }

    @Override
    public Object invoke(MethodInvocation invocation) throws Throwable {
        TraceStatus status = null;

        try{
            Method method = invocation.getMethod();
            // OrderRepository.request() 처럼 나온다. 호출한 클래스.호출한 메소드명
            String message = method.getDeclaringClass().getSimpleName() + "." + method.getName() + "()";
            status =  logTrace.begin(message);
            // target 호출
            Object result = invocation.proceed();
            logTrace.end(status);
            return result;
        }catch(Exception e){
            logTrace.exception(status, e);
            throw e;
        }
    }
}
```

- 다음과 같이, Configuration 통해서 Advice 적용할 수 있다.

```java
@Slf4j
@Configuration
public class ProxyFactoryConfigV1 {

    @Bean
    public OrderRepositoryV1 orderRepositoryV1(LogTrace logTrace){
        OrderRepositoryV1 orderRepository = new OrderRepositoryV1Impl();
        ProxyFactory factory = new ProxyFactory(orderRepository);

        factory.addAdvisor(getAdvisor(logTrace));
        OrderRepositoryV1 proxy = (OrderRepositoryV1) factory.getProxy();
        log.info("ProxyFactory proxy = {} , target = {}", proxy.getClass(), orderRepository.getClass());
        return proxy;
    }

    @Bean
    public OrderServiceV1 orderServiceV1(LogTrace logTrace){
        OrderServiceV1 orderService = new OrderServiceV1Impl(orderRepositoryV1(logTrace));
        ProxyFactory factory = new ProxyFactory(orderService);

        factory.addAdvisor(getAdvisor(logTrace));
        OrderServiceV1 proxy = (OrderServiceV1) factory.getProxy();
        log.info("ProxyFactory proxy = {} , target = {}", proxy.getClass(), orderService.getClass());
        return proxy;
    }

    @Bean
    public OrderControllerV1 orderControllerV1(LogTrace logTrace){
        OrderControllerV1 orderController = new OrderControllerV1Impl(orderServiceV1(logTrace));
        ProxyFactory factory = new ProxyFactory(orderController);

        factory.addAdvisor(getAdvisor(logTrace));
        OrderControllerV1 proxy = (OrderControllerV1) factory.getProxy();
        log.info("ProxyFactory proxy = {} , target = {}", proxy.getClass(), orderController.getClass());
        return proxy;
    }

    private Advisor getAdvisor(LogTrace logTrace){
        //pointcut
        NameMatchMethodPointcut pointcut = new NameMatchMethodPointcut();
        // 아래와 같은 Method Pattern 에만 적용
        pointcut.setMappedNames("request*", "order*", "save*");

        LogTraceAdvice advice = new LogTraceAdvice(logTrace);
        return new DefaultPointcutAdvisor(pointcut, advice);
    }
}
```

# 10. 정리

- Proxy 팩토리를 통해
    - 어떤 부가 기능을
    - 어디에 적용할지
    - 편하게 정의 가능했다.

- 하지만 아직 문제 남아 있다!
    1. Configuration 파일이 지나치게 복잡하다.
        - 만약 Bean이 100개 있다면, 100개 전부 다 Configuration 만들어줘야 한다.
        - 설정 지옥(...)
    2. Component Scan시 적용이 불가능하다.
        - Component Scan 사용하면, 실제 객체를 바로 Bean으로 등록한다.
        - 따라서 다른 방법이 필요하다!

- 이를 해결하기 위해, 이후 Bean 후처리기가 필요하다.
