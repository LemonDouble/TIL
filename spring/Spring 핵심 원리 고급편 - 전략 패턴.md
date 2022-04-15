# Spring 핵심 원리 고급편 - 전략 패턴

# 1. 전략 패턴 - 시작

- 새로운 test 코드 추가

# 2. 전략 패턴 - 예제 1

- 전략 패턴은
    1. 변하지 않는 부분을 Context 라는 곳에 두고
    2. 변하는 부분을 Strategy라는 Interface를 만들고
    3. 해당 Interface를 구현하도록 해 문제를 해결
    - 상속이 아닌 위임(delegate) 로 문제 해결
    
    ![Untitled](https://github.com/LemonDouble/TIL/blob/main/spring/image/Untitled2.png)
    

- 다음과 같이 구현
1. Interface를 정의

```java
public interface Strategy {
    void call();
}
```

1. Context를 정의

```java
/**
 * 필드에 전략을 보관하는 방식
 */
@Slf4j
public class ContextV1 {
    private Strategy strategy;

    public ContextV1(Strategy strategy){
        this.strategy = strategy;
    }

    public void execute(){
        long startTime = System.currentTimeMillis();
        // 비즈니스 로직 실행
        strategy.call(); //위임
        // 비즈니스 로직 종료
        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;
        log.info("resultTime={} ", resultTime);
    }
}
```

1. 이후 Strategy를 구현
2. 다음과 같이 사용!

```java
/**
 * 전략 패턴 사용
 */
@Test
public void strategyV1(){
    StrategyLogic1 strategyLogic1 = new StrategyLogic1();
    // 전략 주입
    ContextV1 context1 = new ContextV1(strategyLogic1);
    context1.execute();

    StrategyLogic2 strategyLogic2 = new StrategyLogic2();
    ContextV1 context2 = new ContextV1(strategyLogic2);
    context2.execute();
}
```

- 전략 패턴의 핵심
    - 구체 클래스가 아닌, Interface에 의존
    - 따라서 Strategy 인터페이스의 구현체가 추가되더라도, Context 코드에는 영향 주지 않음
    - **Spring에서 의존관계 주입에 사용하는 방식이 바로 전략 패턴!**

- 전략 패턴에서 Interface에 의해 느슨하게 연결되어 있기 때문에
    - 템플릿 메서드 패턴에선 부모가 바뀌면 영향받았지만, 전략 패턴은 영향받지 않음
    - Interface만 변화하지 않는다면, 서로 영향을 주지 않음!

# 3. 전략 패턴 - 예제 2

- 마찬가지로, 전략 패턴도 익명 내부 클래스로 사용 가능하다!

```java

@Test
public void strategyV2(){
    Strategy strategyLogic1 = new Strategy(){
        @Override
        public void call() {
            log.info("비즈니스 로직 1 실행");
        }
    };
    ContextV1 context1 = new ContextV1(strategyLogic1);
    context1.execute();

    Strategy strategyLogic2 = new Strategy(){
        @Override
        public void call() {
            log.info("비즈니스 로직 2 실행");
        }
    };
    ContextV1 context2 = new ContextV1(strategyLogic2);
    context2.execute();
}
```

- 더 줄일수도 있다 (Lambda 이용)

```java
@Test
public void strategyV4(){
    
    ContextV1 context1 = new ContextV1(()-> log.info("비즈니스 로직 1 실행"));
    context1.execute();
    ContextV1 context2 = new ContextV1(() -> log.info("비즈니스 로직 2 실행"));
    context2.execute();

}
```

- 익명 내부 클래스를 Java 8부터 지원하는 Lambda로 바꿀 수 있다.
    - Interface에 메서드가 딱 1개만 있으면 사용 가능
    - Strategy Interface는 메서드가 1개만 있으므로, Lambda로 사용 가능하다.

### 선 조립, 후 실행

- 현재 Context 내부에 Strategy Field를 두고 사용하는 중 (ContextV1 코드를 보자)
    - 이 방식은, Context와 Strategy를 실행 전에 미리 조립해 둔 뒤 실행하는 방식이면 유용하다.
    - 한번 조립 뒤에는 단순히 context.execute()만 실행하면 된다!
    - Spring에서, 어플리케이션 로딩 시점에 DI를 끝내고, 실제 요청을 처리하는 것과 같은 원리

- 하지만 단점도 있다.
    - Context와 Strategy를 한번 조립한 뒤에는 전략을 번경하기 힘들다.
    - Context에 setter를 제공하면, 전략을 변경할 순 있다.
        - 하지만 이런 경우, 동시성 이슈 등이 생길 수 있다.
    - 차라리 이런 경우, 새로운 Context를 하나 더 만들고 다른 Strategy를 주입하는 것이 나을 수도 있다.

# 4. 전략 패턴 - 예제 3 ( 더 유연한 전략 패턴 )

- 이전에는 Context 내부에 전략 Field를 만들고, 생성자로 주입받아 사용했다.
    - 이런 방식 대신, Parameter로 생성자를 전달하면?

1. 파라미터로 전략을 받는다.

```java
@Slf4j
public class ContextV2 {

    public ContextV2() {
    }

    public void execute(Strategy strategy){
        long startTime = System.currentTimeMillis();
        // 비즈니스 로직 실행
        strategy.call(); //위임
        // 비즈니스 로직 종료
        long endTime = System.currentTimeMillis();
        long resultTime = endTime - startTime;
        log.info("resultTime={} ", resultTime);
    }
}
```

1. 실행할 때마다 전략을 전달하면 조금 더 유연하게 사용할 수 있다.

```java
@Test
public void strategyV2(){
    ContextV2 context = new ContextV2();
    context.execute(new StrategyLogic1());
    context.execute(new StrategyLogic2());
}
```