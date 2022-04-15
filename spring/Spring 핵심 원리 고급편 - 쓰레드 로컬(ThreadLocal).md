# Spring 핵심 원리 고급편 - 쓰레드 로컬(ThreadLocal)

# 1. 필드 동기화 - 개발

- 기존 로그 추적기는 직접 Paramter를 넘겨줘야 했다.
    - 로그 출력하는 모든 메서드에 Level, ID 가진 TraceId 넘겨주는건 비효율적

- 따라서, Logger가 State를 직접 가지는 식으로 사용해 보자
    - 동시성 문제 생기겠지만 이건 나중에..

- Interface 정의하고, syncTraceId, releaseTraceId 함수 추가 (코드 참고)

- Test 이후 정상작동 확인

# 2. 필드 동기화 - 적용

- 실제로 적용

# 3. 필드 동기화

- 배포하면 동시성 관련 문제가 생긴다!

- 1초 안에 2번/3번 호출하면 다음과 같이 로그가 꼬인다!!
    - nio-exec-7 에서 숫자는 쓰레드 번호

```java
2022-04-15 13:50:07.724  INFO 18016 --- [nio-8080-exec-6] h.advanced.trace.logtrace.FieldLogTrace  : [09a8a31a] OrderController.request()
2022-04-15 13:50:07.724  INFO 18016 --- [nio-8080-exec-6] h.advanced.trace.logtrace.FieldLogTrace  : [09a8a31a] |-->OrderService.request()
2022-04-15 13:50:07.724  INFO 18016 --- [nio-8080-exec-6] h.advanced.trace.logtrace.FieldLogTrace  : [09a8a31a] |  |-->OrderRepository.request()
2022-04-15 13:50:07.848  INFO 18016 --- [nio-8080-exec-7] h.advanced.trace.logtrace.FieldLogTrace  : [09a8a31a] |  |  |-->OrderController.request()
2022-04-15 13:50:07.849  INFO 18016 --- [nio-8080-exec-7] h.advanced.trace.logtrace.FieldLogTrace  : [09a8a31a] |  |  |  |-->OrderService.request()
2022-04-15 13:50:07.849  INFO 18016 --- [nio-8080-exec-7] h.advanced.trace.logtrace.FieldLogTrace  : [09a8a31a] |  |  |  |  |-->OrderRepository.request()
2022-04-15 13:50:07.962  INFO 18016 --- [nio-8080-exec-8] h.advanced.trace.logtrace.FieldLogTrace  : [09a8a31a] |  |  |  |  |  |-->OrderController.request()
2022-04-15 13:50:07.962  INFO 18016 --- [nio-8080-exec-8] h.advanced.trace.logtrace.FieldLogTrace  : [09a8a31a] |  |  |  |  |  |  |-->OrderService.request()
2022-04-15 13:50:07.962  INFO 18016 --- [nio-8080-exec-8] h.advanced.trace.logtrace.FieldLogTrace  : [09a8a31a] |  |  |  |  |  |  |  |-->OrderRepository.request()
2022-04-15 13:50:08.106  INFO 18016 --- [nio-8080-exec-9] h.advanced.trace.logtrace.FieldLogTrace  : [09a8a31a] |  |  |  |  |  |  |  |  |-->OrderController.request()
2022-04-15 13:50:08.107  INFO 18016 --- [nio-8080-exec-9] h.advanced.trace.logtrace.FieldLogTrace  : [09a8a31a] |  |  |  |  |  |  |  |  |  |-->OrderService.request()
2022-04-15 13:50:08.107  INFO 18016 --- [nio-8080-exec-9] h.advanced.trace.logtrace.FieldLogTrace  : [09a8a31a] |  |  |  |  |  |  |  |  |  |  |-->OrderRepository.request()
2022-04-15 13:50:08.205  INFO 18016 --- [io-8080-exec-10] h.advanced.trace.logtrace.FieldLogTrace  : [09a8a31a] |  |  |  |  |  |  |  |  |  |  |  |-->OrderController.request()
2022-04-15 13:50:08.205  INFO 18016 --- [io-8080-exec-10] h.advanced.trace.logtrace.FieldLogTrace  : [09a8a31a] |  |  |  |  |  |  |  |  |  |  |  |  |-->OrderService.request()
2022-04-15 13:50:08.205  INFO 18016 --- [io-8080-exec-10] h.advanced.trace.logtrace.FieldLogTrace  : [09a8a31a] |  |  |  |  |  |  |  |  |  |  |  |  |  |-->OrderRepository.request()
2022-04-15 13:50:08.739  INFO 18016 --- [nio-8080-exec-6] h.advanced.trace.logtrace.FieldLogTrace  : [09a8a31a] |  |<--OrderRepository.request() time=1015ms
2022-04-15 13:50:08.739  INFO 18016 --- [nio-8080-exec-6] h.advanced.trace.logtrace.FieldLogTrace  : [09a8a31a] |<--OrderService.request() time=1015ms
2022-04-15 13:50:08.739  INFO 18016 --- [nio-8080-exec-6] h.advanced.trace.logtrace.FieldLogTrace  : [09a8a31a] OrderController.request() time=1015ms
2022-04-15 13:50:08.850  INFO 18016 --- [nio-8080-exec-7] h.advanced.trace.logtrace.FieldLogTrace  : [09a8a31a] |  |  |  |  |<--OrderRepository.request() time=1001ms
2022-04-15 13:50:08.850  INFO 18016 --- [nio-8080-exec-7] h.advanced.trace.logtrace.FieldLogTrace  : [09a8a31a] |  |  |  |<--OrderService.request() time=1001ms
2022-04-15 13:50:08.850  INFO 18016 --- [nio-8080-exec-7] h.advanced.trace.logtrace.FieldLogTrace  : [09a8a31a] |  |  |<--OrderController.request() time=1002ms
2022-04-15 13:50:08.976  INFO 18016 --- [nio-8080-exec-8] h.advanced.trace.logtrace.FieldLogTrace  : [09a8a31a] |  |  |  |  |  |  |  |<--OrderRepository.request() time=1014ms
2022-04-15 13:50:08.976  INFO 18016 --- [nio-8080-exec-8] h.advanced.trace.logtrace.FieldLogTrace  : [09a8a31a] |  |  |  |  |  |  |<--OrderService.request() time=1014ms
2022-04-15 13:50:08.976  INFO 18016 --- [nio-8080-exec-8] h.advanced.trace.logtrace.FieldLogTrace  : [09a8a31a] |  |  |  |  |  |<--OrderController.request() time=1014ms
2022-04-15 13:50:09.118  INFO 18016 --- [nio-8080-exec-9] h.advanced.trace.logtrace.FieldLogTrace  : [09a8a31a] |  |  |  |  |  |  |  |  |  |  |<--OrderRepository.request() time=1011ms
2022-04-15 13:50:09.118  INFO 18016 --- [nio-8080-exec-9] h.advanced.trace.logtrace.FieldLogTrace  : [09a8a31a] |  |  |  |  |  |  |  |  |  |<--OrderService.request() time=1011ms
2022-04-15 13:50:09.118  INFO 18016 --- [nio-8080-exec-9] h.advanced.trace.logtrace.FieldLogTrace  : [09a8a31a] |  |  |  |  |  |  |  |  |<--OrderController.request() time=1012ms
2022-04-15 13:50:09.213  INFO 18016 --- [io-8080-exec-10] h.advanced.trace.logtrace.FieldLogTrace  : [09a8a31a] |  |  |  |  |  |  |  |  |  |  |  |  |  |<--OrderRepository.request() time=1008ms
2022-04-15 13:50:09.213  INFO 18016 --- [io-8080-exec-10] h.advanced.trace.logtrace.FieldLogTrace  : [09a8a31a] |  |  |  |  |  |  |  |  |  |  |  |  |<--OrderService.request() time=1008ms
2022-04-15 13:50:09.213  INFO 18016 --- [io-8080-exec-10] h.advanced.trace.logtrace.FieldLogTrace  : [09a8a31a] |  |  |  |  |  |  |  |  |  |  |  |<--OrderController.request() time=1008ms
```

- Race Condition, Critical Section으로 인해 문제 발생
    - Singleton이기 때문에, 문제 발생
    - 이런 문제를 해결하기 위해 ThreadLocal 사용

- TIP!
    - 지역변수에서는 동시성 문제 X, 쓰레드마다 다른 메모리 할당됨
    - 보통 같은 Instance, Static같은 공용 Field에 접근할떄 발생.
    - CR만 하면 발생 X, Update하기 때문에 발생한다.

# 4. ThreadLocal - 소개

- 한 Thread만 접근 가능한 특별한 저장소

- ThreadLocal은 제네릭 지원
    - 저장 : ThreadLocal.set(something);
    - 조회 : ThreadLocal.get();
    - 제거 : ThreadLocal.remove();
- 주의점:
    - 해당 쓰레드가 ThreadLocal 사용한 뒤에, remove() 호출하여 제거해줘야 한다!

```java
@Slf4j
public class ThreadLocalService {

		// 이런 식으로 간단하게 사용!
    private ThreadLocal<String> nameStore = new ThreadLocal<>();

    public String logic(String name){
        log.info("저장 name={} -> nameStore{}", name, nameStore.get());
        nameStore.set(name);

        // 저장하는데 1초쯤 걸린다고 가정
        sleep(1000);

        log.info("조회 nameStore={}", nameStore.get());
        return nameStore.get();
    }

    private void sleep(int millis){
        try{
            Thread.sleep(millis);
        }catch(InterruptedException e){
            e.printStackTrace();
        }
    }
}
```

# 5. Thread Local - 개발

- FieldLogTrace의 State Field를 ThreadLocal로 변경 (코드 참조)

# 6. Thread Local - 적용

- Interface를 기반으로 만들어서, Client 코드 변경 없이 변경 가능하다!
- LogTraceConfig 에서 아래처럼 바꿔주면 전체 변경 끝

```java
@Configuration
public class LogTraceConfig {

    @Bean
    public LogTrace logTrace(){
        return new ThreadLocalLogTrace();
    }
}
```

# 7. Thread Local - 주의사항

- Thread Local 사용 후, remove(제거) 호출하지 않으면
    - WAS(Tomcat)처럼 쓰레드 풀 사용하는 경우,심각한 문제 발생 가능

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/fe82fe45-b00c-48bb-a81f-88d81ce6da67/Untitled.png)

- 이전 사용자(A)가 A thread 이용해 Thread Local 사용하고, remove 하지 않으면..
    - 사용자 A의 데이터가 thread Local A 값에 저장된채로 유지된다.
    - 따라서, 다음 사용자가 이전 사용자의 Data를 받을 수도 있다!!!!

- 대참사를 피하려면 ThreadLocal.remove()를 통해 꼭 제거하자.
    - Filter나 Interceptor 통해 제거하거나, 직접 제거하거나