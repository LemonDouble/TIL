# 1. 템플릿 메서드 패턴 - 시작

- 앞에서 만든 로그 추적기를 도입하려니, 코드가 굉장히 지저분해진다!

- 도입 전

```java
@GetMapping("/v0/request")
public String request(String itemId) {
 orderService.orderItem(itemId);
 return "ok";
}
```

- 도입 후

```java
@GetMapping("/v3/request")
public String request(String itemId) {
	 TraceStatus status = null;
	 try {
		 status = trace.begin("OrderController.request()");
		 orderService.orderItem(itemId); //핵심 기능
		 trace.end(status);
	 } catch (Exception e) {
		 trace.exception(status, e);
		 throw e;
	 }
	 return "ok";
}
```

- 이 때, 핵심 기능은 orderService를 호출하고, Return 하는 부분
- 이 때, 부가 기능은 Logging, Transaction 등의 부가적인 기능 (핵심 기능을 보조함)

- 따라서, 도입 후에는 핵심 기능과 부가 기능이 섞여있다.

### 어떻게 해결할 수 있을까?

- 아래와 같은 코드가 항상 반복됨을 볼 수 있다.

```java
TraceStatus status = null;
	 try {
		 status = trace.begin("OrderController.request()");
		 //핵심 기능은 여기에 작성
		 trace.end(status);
	 } catch (Exception e) {
		 trace.exception(status, e);
		 throw e;
	 }
	 return "ok";
```

- 하지만 핵심 로직이 코드 중앙에 있어, 간단히 메소드로 추출하기 힘들다.
    - Try-catch도 있어서 추출이 힘들다.

### 템플릿 메서드 패턴 (Template Method Pattern)은 이런 문제를 해결하기 위한 디자인 패턴이다!

# 2. 템플릿 메서드 패턴 - 예제 1

```java
private void logic1(){
        long startTime = System.currentTimeMillis();
        // 비즈니스 로직 실행
        log.info("비즈니스 로직1 실행");
        // 비즈니스 로직 종료
        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;
        log.info("resultTime={} ", resultTime);
    }

    private void logic2(){
        long startTime = System.currentTimeMillis();
        // 비즈니스 로직 실행
        log.info("비즈니스 로직2 실행");
        // 비즈니스 로직 종료
        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;
        log.info("resultTime={} ", resultTime);
    }
```

- 두 코드는 비즈니스 로직 빼고 다른 부분이 정확히 같다! 즉,
    - 바뀌는 부분 : 비즈니스 로직
    - 바뀌지 않는 부분 :  시간을 측정하는 부분

- 따라서 이를 분리하고 싶다!

# 3. 템플릿 메서드 패턴 - 예제 2

- 다음과 같은 추상 메서드를 이용하여 해결!
    - 바뀌는 부분은, 상속받아 이용하도록 한다.
    - 바뀌지 않는 부분은, execute()에 구현되어 있다.

```java
@Slf4j
public abstract class AbstractTemplate {

    public void execute(){
        long startTime = System.currentTimeMillis();
        // 비즈니스 로직 실행
        call();
        // 비즈니스 로직 종료
        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;
        log.info("resultTime={} ", resultTime);
    }

    // 자식 클래스를 만들어서 call을 구현하도록 해결!
    protected abstract void call();
}

// 상속받아 call을 오버라이딩
@Slf4j
public class SubClassLogic1 extends AbstractTemplate{

    @Override
    protected void call() {
        log.info("비즈니스 로직 1 실행");
    }
}
```

- 이후 다음과 같이 사용

```java
@Test
public void templateMethodV1(){
    AbstractTemplate template1 = new SubClassLogic1();
    template1.execute();
    AbstractTemplate template2 = new SubClassLogic2();
    template2.execute();
}
```

- execute 실행 도중, call()을 만났을 때 오버라이딩 되어 있으므로, 자식 클래스의 로직이 실행된다.

- 템플릿 메서드 패턴은 다형성을 이용해, 변하는 부분과 변하지 않는 부분을 분리한다.

# 4. 템플릿 메서드 패턴 - 예제 3

- 템플릿 메서드 패턴은 abstract class를 상속받는 자식 클래스를 계속 만들어야 하는 단점이 있다.
    - 위의 SubClassLogic1() 같은 거..

- 익명 내부 클래스를 사용하면 이를 보완할 수 있다.
    - 객체 인스턴스를 생성하면서, 바로 상속받는 자식 클래스를 정의할 수 있다.
    
- 익명 내부 클래스 적용

```java
@Test
public void templateMethodV2(){
    AbstractTemplate template1 = new AbstractTemplate() {
        @Override
        protected void call() {
            log.info("비즈니스 로직 1 실행");
        }
    };
    template1.execute();

    AbstractTemplate template2 = new AbstractTemplate() {
        @Override
        protected void call() {
            log.info("비즈니스 로직 2 실행");
        }
    };
    template2.execute();
}
```

# 5. 템플릿 메서드 패턴 - 적용 1

- 기존 Controller는 다음과 같이 변경된다.
    - 참고로, Generic은 Primitive type은 반환하지 못 한다!
    - 따라서, int 대신 Integer, void 대신 Void 반환하고, Void의 경우 null 반환하자.

```java
AbstractTemplate<String> template = new AbstractTemplate<String>(trace) {
    @Override
    protected String call() {
        orderService.orderItem(itemId);
        return "ok";
    }
};
return template.execute("OrderController.request()");
```

# 6. 템플릿 메서드 패턴 - 적용 2

- 좋은 설계란?
    - 기준이 여러가지 있을 수 있다.
    - 강의자의 생각은 “변경” 이 일어날때 드러남.
    - 변경이 일어났을 때, 변화가 최소화될수록 좋다.

- Template Method를 이용하여, 요구사항이 변경되더라도 Template만 고치면 된다!

# 7. 템플릿 메서드 패턴 - 정의

- Template Method Pattern의 목적은 다음과 같다.
    - 작업에서 알고리즘의 골격을 정의
    - 일부 단계를 하위 클래스로 연기
    - 하위 클래스가 알고리즘 구조를 변경하지도 않고, 특정 단계를 확장 가능

- 하지만 Template Method Pattern은 상속을 사용한다.
    - 따라서 상속에서 오는 단점을 그대로 가져간다.
    - 자식 클래스가, 부모 클래스와 강결합 되는 문제가 있다.
    - 자식 클래스는 부모 클래스의 기능을 하나도 사용하지 않지만, 부모 클래스를 상속받는 중

- 상속을 받는다 = 부모 클래스에 의존한다.
    - extends 다음 부모의 구체 클래스에 강력하게 의존한다.
    - 즉, 부모 클래스에서 뭔가 바뀌면 자식 클래스 또한 영향을 받는다!

- 또한, 결국 상속을 사용하므로, 별도의 클래스/ 익명 내부 클래스를 만들어야 한다.
    - 이를 개선하기 위한 패턴이 전략 패턴 (Strategy Pattern)이다.