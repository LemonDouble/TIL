# Spring 핵심 원리 고급편 - 템플릿 콜백 패턴

# 1. 템플릿 콜백 패턴 - 시작

- Callback : 다른 코드의 Parameter로 넘겨주는 실행 가능한 코드
    - Callback을 넘겨받는 코드는, 이 코드를 직접 실행할 수도
    - 나중에 실행할 수도 있다.

- 앞의 전략 패턴에서, 콜백은 Strategy!

- Java의 Callback
    - Java에서 실행 가능한 코드를 넘기려면 객체가 필요
    - Java 8 이전에는 메서드가 하나 뿐인 Interface를 구현, 익명 내부 클래스를 사용했음
    - Java 8 이후부터는 Lambda를 사용!

### 템플릿 콜백 패턴

- GoF 패턴 아님!
    - Spring에서 자주 사용해서, 스프링에서만 이렇게 부른다.
    - Spring 내부에서 ~Template 라는 메소드 있으면, 템플릿 콜백 패턴이라고 생각하면 됨
        - JdbcTemplate, RestTemplate, TransactionTemplate 등..
        

# 2. 템플릿 콜백 패턴 - 예제

- Callback Interface

```java
public interface Callback {
    void call();
}
```

- Template Class

```java
@Slf4j
public class TimeLogTemplate {

    public void execute(Callback callback){
        long startTime = System.currentTimeMillis();
        //비즈니스 로직 실행
        callback.call();
        //비즈니스 로직 종료
        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;
        log.info("resultTime={}", resultTime);
   }
}
```

# 3. 템플릿 콜백 패턴 - 적용

1. Interface는 Generic 이용하여 여러 Return Type 지원하도록 정의

```java
// 여러 반환 타입 지원하기 위해 Generic 사용
public interface TraceCallback <T>{
    T call();
}
```

1. Template 또한 Generic 이용하여 작성. 여러 Return Type 지원

```java
public class TraceTemplate {
    private final LogTrace trace;

    public TraceTemplate(LogTrace trace) {
        this.trace = trace;
    }

    public <T> T execute (String message, TraceCallback<T> callback){
        TraceStatus status = null;
        try{
            status = trace.begin(message);
            // 비즈니스 로직 호출
            T result = callback.call();
            trace.end(status);
            return result;
        }catch (Exception e){
            trace.exception(status, e);
            throw e;
        }
    }
}
```

1. 이후 다음과 같이 사용
    - 첫 인자는 함수 이름
    - 두번쨰 인자는 비즈니스 로직

```java
@GetMapping("/v5/request")
    public String request(String itemId){
				
        return template.execute("OrderController.request()", new TraceCallback<String>() {
            @Override
            public String call() {
                orderService.orderItem(itemId);
                return "ok";
            }
        });

    }
```

# 4. 템플릿 콜백 패턴 - 정리

- Template Callback Pattern 이용하여
    - 변하는 코드와 변하지 않는 코드를 분리
    - Lambda를 이용해 간편하게 사용 가능하도록 변환

- 하지만 한계 존재한다.
    - 결국 원본 코드를 수정해야 한다.
    - 아무리 간편하게 만들어도, 본질적으로 여러 코드를 다 수정해야 한다!
    - 이를 해결하기 위해 Proxy라는 개념이 나왔다.

- Tip : 템플릿 콜백 패턴은 실제로 Spring에서 사용되는 방식
    - ~~Template 라는 클래스 만나면, 위와 같이 구현되어 있다고 보면 된다.