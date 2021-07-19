# Spring 핵심 원리 : 빈 스코프

분류: Spring
작성일시: 2021년 7월 19일 오후 3:10

## 1. 빈 스코프란?

- Bean Scope : 빈이 존재할 수 있는 범위를 뜻한다.

- Spring이 지원하는 Scope
    1. Singleton : 기본 Scope, 스프링 컨테이너의 시작부터 종료까지 유지되는 가장 넓은 범위의 Scope
    2. Prototype : 스프링 컨테이너는 Bean의 생성/의존관계 주입까지만 관여하고, 더는 관리하지 않는 매우 짧은 범위의 스코프이다. (종료 메서드는 호출되지 않는다)
    3. 웹 관련 Scope
        - Request : 웹 요청이 들어오고, 나갈 떄까지 유지되는 Scope
        - Session : 웹 세션이 생성되고, 종료될 때 까지 유지되는 Scope
        - Application : 웹의 서블릿 컨텍스트와 같은 범위로 유지되는 Scope

- Bean Scope의 등록 방법
    1. Component Scan

    ```java
    @Scope("prototype")
    @Component
    public class HelloBean {
    ...
    }
    ```

    1. Bean 수동 등록

    ```java
    @Scope("prototype")
    @Bean
    PrototypeBean HelloBean(){
    	return new HelloBean();
    }
    ```

## 2. Prototype 스코프

- Singleton Scope의 빈을 조회하면, 항상 같은 인스턴스의 Spring Bean을 반환한다.
- Prototype Scope의 빈을 조회하면, 항상 새로운 인스턴스를 생성해서 반환한다.

![Spring%20%E1%84%92%E1%85%A2%E1%86%A8%E1%84%89%E1%85%B5%E1%86%B7%20%E1%84%8B%E1%85%AF%E1%86%AB%E1%84%85%E1%85%B5%20%E1%84%87%E1%85%B5%E1%86%AB%20%E1%84%89%E1%85%B3%E1%84%8F%E1%85%A9%E1%84%91%E1%85%B3%201bdbb20920d14453a80afaf30c229dc7/Untitled.png](https://github.com/LemonDouble/TIL/blob/main/spring/img/Untitled%2018.png)

- 핵심은, Prototype Bean의 경우 Spring은 생성, DI, 초기화까지만 처리한다는 점!
- Container는 Prototype Bean을 관리하지 않으므로, Bean을 관리할 책임은 Client에 있다.
- 따라서, @PreDestroy와 같은 종료 메서드는 호출되지 않는다.

- SigletonBean의 경우, init과 Destroy는 한번만 호출되고, 두 빈은 같다

    (test/../scope/SingletonTest.class)

```java
package hello.core.scope;

import org.assertj.core.api.Assertions;
import org.junit.jupiter.api.Test;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Scope;

import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;

public class SingletonTest {

    @Test
    void singletonBeanFind(){
       AnnotationConfigApplicationContext ac =  new AnnotationConfigApplicationContext(singletonBean.class);

        singletonBean singletonBean1 = ac.getBean(singletonBean.class);
        singletonBean singletonBean2 = ac.getBean(singletonBean.class);

        System.out.println("singletonBean1 = " + singletonBean1);
        System.out.println("singletonBean2 = " + singletonBean2);
        Assertions.assertThat(singletonBean1).isSameAs(singletonBean2);

        ac.close();
    }

    @Scope("singleton")
    static class singletonBean{
        @PostConstruct
        public void init(){
            System.out.println("SingletonBean.init");
        }

        @PreDestroy
        public void destroy(){
            System.out.println("SingletonBean.destroy");
        }

    }
}
```

- 결과

```java
SingletonBean.init
singletonBean1 = hello.core.scope.SingletonTest$singletonBean@35645047
singletonBean2 = hello.core.scope.SingletonTest$singletonBean@35645047
SingletonBean.destroy
```

---

- PrototypeBean의 경우, init만 호출되고 Destroy는 호출되지 않는다.

    (test/../scope/PrototypeTest.class)

    ```java
    package hello.core.scope;

    import org.assertj.core.api.Assertions;
    import org.junit.jupiter.api.Test;
    import org.springframework.context.annotation.AnnotationConfigApplicationContext;
    import org.springframework.context.annotation.Scope;

    import javax.annotation.PostConstruct;
    import javax.annotation.PreDestroy;

    public class PrototypeTest {

        @Test
        void PrototypeBeanFind(){
           AnnotationConfigApplicationContext ac =  new AnnotationConfigApplicationContext(PrototypeBean.class);

            //호출 직전에 생성된다.
            System.out.println("find prototypeBean1");
            PrototypeBean PrototypeBean1 = ac.getBean(PrototypeBean.class);
            System.out.println("find prototypeBean2");
            PrototypeBean PrototypeBean2 = ac.getBean(PrototypeBean.class);

            System.out.println("PrototypeBean1 = " + PrototypeBean1);
            System.out.println("PrototypeBean2 = " + PrototypeBean2);
            //Bean 둘은 다르다.
            Assertions.assertThat(PrototypeBean1).isNotSameAs(PrototypeBean2);

    				//종료 메서드 호출되지 않으므로, 만약 종료 메서드 필요하다면
    				//Client가 직접 호출해줘야 한다.
    				//PrototypeBean1.destroy();
    				//PrototypeBean2.destroy();
    			
            ac.close();
        }

        @Scope("prototype")
        static class PrototypeBean{
            @PostConstruct
            public void init(){
                System.out.println("PrototypeBean.init");
            }

            @PreDestroy
            public void destroy(){
                System.out.println("PrototypeBean.destroy");
            }

        }
    }
    ```

- 결과

```java
find prototypeBean1
PrototypeBean.init
find prototypeBean2
PrototypeBean.init
// 호출 직전에 생성되고, 두번 생성된다.
PrototypeBean1 = hello.core.scope.PrototypeTest$PrototypeBean@35645047
PrototypeBean2 = hello.core.scope.PrototypeTest$PrototypeBean@6f44a157
// 두 빈은 다르다.
```

- Prototype 빈의 특징 정리
    1. Spring 컨테이너에 요청할 때 마다 새로 생성된다.
    2. Spring 컨테이너는 생성, DI, 초기화까지만 관여한다.
    3. 종료 메서드는 호출되지 않는다.
    4. 따라서, 프로토타입 빈은 직접 프로토타입 빈을 조회한 클라이언트가 관리해야 한다.

## 3. Prototype Scope - Singleton Bean과 함께 사용시의 문제점

- Prototype Bean을 요청하면 항상 새로운 객체 인스턴스를 생성해서 반환하지만, Singleton Bean과 함께 사용할 때는 의도한 대로 잘 동작하지 않는다.

- 프로토타입 빈을 직접 요청하는 예제

![Spring%20%E1%84%92%E1%85%A2%E1%86%A8%E1%84%89%E1%85%B5%E1%86%B7%20%E1%84%8B%E1%85%AF%E1%86%AB%E1%84%85%E1%85%B5%20%E1%84%87%E1%85%B5%E1%86%AB%20%E1%84%89%E1%85%B3%E1%84%8F%E1%85%A9%E1%84%91%E1%85%B3%201bdbb20920d14453a80afaf30c229dc7/Untitled%201.png](https://github.com/LemonDouble/TIL/blob/main/spring/img/Untitled%2019.png)

- 코드로 확인 (test/.../scope/SingletonWithPrototypeTest1.class)

```java
package hello.core.scope;

import org.assertj.core.api.Assertions;
import org.junit.jupiter.api.Test;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Scope;

import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;

public class SingletonWithPrototypeTest1 {

    @Test
    void prototypeFind(){
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(PrototypeBean.class);

        PrototypeBean prototypeBean1 = ac.getBean(PrototypeBean.class);
        prototypeBean1.addCount();
        //count = 1
        Assertions.assertThat(prototypeBean1.getCount()).isEqualTo(1);

        PrototypeBean prototypeBean2 = ac.getBean(PrototypeBean.class);
        prototypeBean2.addCount();
        //count = 1
        Assertions.assertThat(prototypeBean2.getCount()).isEqualTo(1);

    }

    @Scope("prototype")
    static class PrototypeBean{
        private int count = 0;

        public void addCount(){
            count++;
        }

        public int getCount(){
            return count;
        }

        @PostConstruct
        public void init(){
            System.out.println("PrototypeBean.init" + this);
        }

        //호출되지 않음!
        @PreDestroy
        public void destroy(){
            System.out.println("PrototypeBean.destroy");
        }
    }
}
```

---

- 싱글톤 빈에서 프로토타입 빈 사용시

![Spring%20%E1%84%92%E1%85%A2%E1%86%A8%E1%84%89%E1%85%B5%E1%86%B7%20%E1%84%8B%E1%85%AF%E1%86%AB%E1%84%85%E1%85%B5%20%E1%84%87%E1%85%B5%E1%86%AB%20%E1%84%89%E1%85%B3%E1%84%8F%E1%85%A9%E1%84%91%E1%85%B3%201bdbb20920d14453a80afaf30c229dc7/Untitled%202.png](https://github.com/LemonDouble/TIL/blob/main/spring/img/Untitled%2020.png)

- 테스트 코드로 확인하기 (test/.../scope/SingletonWithPrototypeTest1.class)

```java
package hello.core.scope;

import org.assertj.core.api.Assertions;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Scope;

import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;

public class SingletonWithPrototypeTest1 {

    @Test
    void prototypeFind(){
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(PrototypeBean.class);

        PrototypeBean prototypeBean1 = ac.getBean(PrototypeBean.class);
        prototypeBean1.addCount();
        //count = 1
        Assertions.assertThat(prototypeBean1.getCount()).isEqualTo(1);

        PrototypeBean prototypeBean2 = ac.getBean(PrototypeBean.class);
        prototypeBean2.addCount();
        //count = 1
        Assertions.assertThat(prototypeBean2.getCount()).isEqualTo(1);
    }

    @Test
    void singletonClientUsePrototype(){
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(ClientBean.class,PrototypeBean.class);

        ClientBean clientBean1 = ac.getBean(ClientBean.class);
        int count1 = clientBean1.logic();
        Assertions.assertThat(count1).isEqualTo(1);

        ClientBean clientBean2 = ac.getBean(ClientBean.class);
        int count2 = clientBean2.logic();
        // 1이 아니라 2!
        Assertions.assertThat(count2).isEqualTo(2);
    }

    @Scope("singleton")
    static class ClientBean{
        //생성 시점에 주입 (1번 주입됨)
        private final PrototypeBean prototypeBean;

        @Autowired
        public ClientBean(PrototypeBean prototypeBean) {
            this.prototypeBean = prototypeBean;
        }

        public int logic(){
            //이떄 prototypeBean은 같은 Bean
            prototypeBean.addCount();
            int count = prototypeBean.getCount();
            return count;
        }
    }

    @Scope("prototype")
    static class PrototypeBean{
        private int count = 0;

        public void addCount(){
            count++;
        }

        public int getCount(){
            return count;
        }

        @PostConstruct
        public void init(){
            System.out.println("PrototypeBean.init" + this);
        }

        //호출되지 않음!
        @PreDestroy
        public void destroy(){
            System.out.println("PrototypeBean.destroy");
        }
    }
}
```

- 이 경우, Prototype Bean은 마치 Singleton Bean처럼, Singleton Bean의 LifECycle과 같이 유지된다.
- 하지만 개발자의 의도는 이것과 다르다. Prototype을 선언했다면, 사용할 떄마다 새로 생성해서 사용하는 것을 원할 것이다.
- 아래의 Provider로 문제를 해결할 수 있다.

- 참고 ) 여러 Bean에서 같은 Prototype Bean을 주입받으면, 주입 받는 시점에 각각 새로운 Prototype Bean이 생성된다.

## 4. 싱글톤 빈과 함께 사용시, Provider로 문제 해결

- Singleton Bean과 Prototype Bean을 같이 사용할 때, 어떻게 하면 사용할 때 마다 새로운 Prototype Bean을 생성할 수 있을까?

- 가장 간단한 방법(무식한 방법)
    - Logic이 실행될 때마다 Spring 컨테이너에 새로 요청하는 방법

    ```java
    @Scope("singleton")
    static class ClientBean{		
    		//ApplicationContext는 Spring에서
    		//Spring Bean을 관리하는 컨테이너라 주입이 가능하다.
    		@Autowired
        private ApplicationContext ac;
        public int logic() {
            PrototypeBean prototypeBean = ac.getBean(PrototypeBean.class);
            prototypeBean.addCount();
            int count = prototypeBean.getCount();
            return count;
        }
    }
    ```

    - 실행시, ac.getBean()을 통해 항상 새로운 프로토타입 빈이 생성된다.
    - 의존관계를 외부에서 주입(DI) 하는게 아니라, 필요한 의존관계를 찾는 것을 Dependency Lookup (DL), 의존관계 조회(탐색) 이라고 한다.
    - 하지만 이렇게 Spring의 ApplicationContext 전체를 주입받게 되면, Spring Container에 종속적인 코드가 되고, 단위 테스트도 어려워진다.
    - 지금 필요한 기능은 필요한 Bean을 컨테이너에서 찾아주는, DL 정도의 기능만 제공하는 뭔가가 있으면 된다. (ApplicationContext 전체 대신)

- ObjectFactory, ObjectProvider
    - 지정한 Bean을 컨테이너에서 대신 찾아주는 DL 서비스를 제공하는 것이 ObjectProvider
    - ObjectFactory + 편의 기능 = > ObjectProvider

    ```java
    		@Scope("singleton")
        static class ClientBean{

            //ObjectProvider 선언, ObjectProvider는 Bean 등록하지 않았지만
    				//Spring이 Bean을 자동으로 등록해서 DI 해 준다.
            @Autowired
            private ObjectProvider<PrototypeBean> prototypeBeanProvider;

            public int logic(){
                PrototypeBean prototypeBean = prototypeBeanProvider.getObject();
                prototypeBean.addCount();;
                int count = prototypeBean.getCount();
                return count;
            }
        }
    ```

    - 실행시 prototypeBeanProvider.getObject(); 통해 항상 새로운 빈이 생성됨
    - ObjectProvider는 Spring Container를 통해 해당 Bean을 찾아서 반환한다.
    - Spring 제공 기능을 사용하지만, 기능이 단순하므로 단위테스트를 만들거나 mock(모조품) 코드를 만들기 훨씬 쉬워진다.

- 특징)
    - ObjectFactory : 기능이 단순, 별도의 라이브러리 필요 없음, Spring에 의존
    - ObjectProvider : ObjectFactory 상속, Stream 처리 등의 편의기능이 많고, 별도의 라이브러리 필요 없음. Spring에 의존

- JSR-330 Provider
    - JSR-330 JAVA 표준을 사용하는 방법
    - JAVA 표준이지만, javax.inject:javax.inject:1 를 추가해줘야 한다.

- build.gradle의 dependencies 안에 다음 코드 추가

```java
dependencies {
			implementation 'javax.inject:javax.inject:1'
			....
}
```

- 이후 Javax의 Provider로 변경

```java
@Scope("singleton")
    static class ClientBean{

        //Javx의 Provider 선언
        @Autowired
        private Provider<PrototypeBean> prototypeBeanProvider;

        public int logic(){
						//Method 이름 : Get
            PrototypeBean prototypeBean = prototypeBeanProvider.get();
            prototypeBean.addCount();;
            int count = prototypeBean.getCount();
            return count;
        }
    }
```

- 위와 마찬가지로, 항상 새로운 Prototype Bean이 생성된다.
- Java 표준이므로, 기능이 단순하므로 단위 테스트를 하거나, mock 코드를 만들기 더 쉬워진다.
- Java 표준이므로, Spring 이외의 다른 컨테이너에서도 사용할 수 있다.

- 정리
    - Prototype Bean을 사용하는 경우 : 사용할 떄마다 의존관계 주입이 완료된 새로운 객체가 필요한 경우
    - 하지만 대부분의 경우, Singleton Bean으로도 대부분의 문제를 해결할 수 있으므로 Prototype Bean을 직접적으로 사용하는 경우는 드물다.
    - ObjectProvier, JSR303 Provider 등은 프로토타입 뿐 아니라, DL이 필요한 경우는 언제든지 사용할 수 있다.

- ObjectProvider vs JSR303 Provider
    - 다른 컨테이너에서도 코드가 동작해야 하는 경우 - JSR303 Provider
    - 그 이외 - Spring에서 제공하는 기능이 더 다양하고 편하므로 그냥 ObjectProvider 쓰자..

        (Spring이 사실상 표준이므로)

## 5. 웹 스코프

- 웹 스코프의 특징
    - 웹 스코프는 웹 환경에서만 동작한다.
    - 웹 스코프는 프로토타입과 다르게, Spring이 해당 스코프의 종료시점까지 관리한다.
    - 따라서, 종료 메서드가 호출된다.

- 웹 스코프의 종류
    - Request : HTTP 요청 하나가 들어오고, 나갈 때까지 유지되는 Scope. 각각의 HTTP 요청마다 별도의 빈 Instance가 생성되고 관리된다.
    - Session : HTTP Session과 동일한 Lifecycle을 가지는 Scope
    - Application : 서블릿 컨텍스트(ServletContext)와 동일한 Lifecycle을 가지는 Scope
    - Websocket : Web Socket이랑 동일한 Lifecycle을 가지는 Scope

![Spring%20%E1%84%92%E1%85%A2%E1%86%A8%E1%84%89%E1%85%B5%E1%86%B7%20%E1%84%8B%E1%85%AF%E1%86%AB%E1%84%85%E1%85%B5%20%E1%84%87%E1%85%B5%E1%86%AB%20%E1%84%89%E1%85%B3%E1%84%8F%E1%85%A9%E1%84%91%E1%85%B3%201bdbb20920d14453a80afaf30c229dc7/Untitled%203.png](https://github.com/LemonDouble/TIL/blob/main/spring/img/Untitled%2021.png)

- 즉, HTTP 요청이 들어오고 나갈 때까진 같은 Bean이 할당된다.

## 6. Request 스코프 예제 만들기

- Web Scope는 웹 환경에서만 동작하므로, Web 환경이 동작하도록 라이브러리를 추가

    (build.gradle에 추가)

    ```java
    dependencies {
    	//Web 라이브러리 추가
    	implementation 'org.springframework.boot:spring-boot-starter-web'
    	...
    }
    ```

- 이후 CoreApplication.class 실행시키면 Tomcat started on port(s): 8080 (http) with context path '' 볼 수 있다.
- 이후 [localhost:8080](http://localhost:8080) 으로 들어가보면 Whitelabel Error Page 나온다. (내장 Tomcat 사용한다)

- 포트를 바꾸려면, main/resources/application.properties에서

---

- Request 스코프 예제 개발
    - 만약 동시에 여러 HTTP 요청이 오면, 정확히 어떤 요청이 남긴 로그인지 구분하기 어렵다.
    - 이럴때 Request 스코프를 사용하면 좋다.

    ```java
    [d06b992f...] request scope bean create
    [d06b992f...][http://localhost:8080/log-demo] controller test
    [d06b992f...][http://localhost:8080/log-demo] service id = testId
    [d06b992f...] request scope bean close
    ```

    - [UUID][requestURL]{message}
    - UUID를 통해서 각 유저를 구분할 수 있다.
    - RequestURL 정보를 통해서 어떤 URL에서 로그가 남았는지 확인할 수 있다.

- src/main/java/common Package 만든 후 MyLogger.class

```java
package hello.core.common;

import org.springframework.context.annotation.Scope;
import org.springframework.stereotype.Component;

import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;
import java.util.UUID;

@Component
//Scope가 request이므로, HTTP request 하나당 생성, 요청이 끝날떄 소멸
@Scope(value= "request")
public class MyLogger {

    private String uuid;
    private String requestURL;

    public void setRequestURL(String requestURL) {
        this.requestURL = requestURL;
    }
    
		//LOG 기능
    public void log(String message){
        System.out.println("[" + uuid + "]" + "[" + requestURL + "] "+ message);
    }
		
		//생성될 때 UUID를 만든다
    @PostConstruct
    public void init(){
        this.uuid = UUID.randomUUID().toString();
        System.out.println("[" + uuid + "] request scope bean create : " + this);
    }

		//Destroy 될 때 알려준다.
    @PreDestroy
    public void close(){
        System.out.println("[" + uuid + "] request scope bean close : " + this);
    }
}
```

- 로그를 출력하기 위한 클래스
- RequestURL은 빈이 생성되는 시점에 알 수 없으므로, 외부에서 Setter로 입력받는다.

- Logger 사용 위해서 Controller 생성 (src/main/java/web/LogDemoController.class)

```java
package hello.core.web;

import hello.core.common.MyLogger;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

import javax.servlet.http.HttpServletRequest;

@Controller
@RequiredArgsConstructor
public class LogDemoController {
    private final LogDemoService logDemoService;
    private final MyLogger myLogger;

    //URL log-demo 라는 요청이 오면 반환
    @RequestMapping("log-demo")
    //view 화면 없이 바로 반환
    @ResponseBody
    public String logDemo(HttpServletRequest request){
        String requestURL = request.getRequestURI().toString();
        myLogger.setRequestURL(requestURL);

        myLogger.log("controller test");
        logDemoService.logic("testId");
        return "OK";
    }
}
```

- 마찬가지로 Service도 추가 (src/main/java/web/LogDemoService.class)

```java
package hello.core.web;

import hello.core.common.MyLogger;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;

@Service
@RequiredArgsConstructor
public class LogDemoService {

    private final MyLogger myLogger;

    public void logic(String id){
        myLogger.log("service id = " + id);
    }
}
```

- 하지만 이렇게 작성할 시, Spring에서 오류가 발생한다.

```java
Error creating bean with name 'myLogger': Scope 'request' is not active for the
current thread; consider defining a scoped proxy for this bean if you intend to
refer to it from a singleton;
```

- Controller, Service가 DI 통해 MyLogger를 주입받아야 하는데, MyLogger의 LifeCycle은 HTTP Request가 생길 때 생성되므로 어플리케이션 실행 시 MyLogger가 없으므로 에러가 발생한다.
- 해결 방법은 바로 밑 Provider 사용을 통해..

참고)

- requestURL을 myLogger에 저장하는 부분 (아래 Code) 는 컨트롤러보다, 공통 처리가 가능한 Spring 인터셉터, 서블릿 필터 등을 사용하는게 좋다.
- 현재 예제에서는 예제 단순화를 위해 Controller에 구현했다.

```java
    public String logDemo(HttpServletRequest request){
        String requestURL = request.getRequestURI().toString();
        myLogger.setRequestURL(requestURL);

        myLogger.log("controller test");
        logDemoService.logic("testId");
        return "OK";
    }
```

- LogDemoService.class 는 Service 계층이다.
- Request Scope의 MyLogger 덕분에, requestURL 같은 비즈니스와 관련없는 부분을 Service 계층에 넘기지 않고 사용할 수 있다.

- 웹과 관련된 부분은 최대 Controller까지만 사용하는 것이 좋다. Service 계층은 위와 같이, 웹 기술에 종속되지 않고 비즈니스 로직만 순수하게 유지하는 것이 유지보수 관점에서 좋다!

## 7. 스코프와 Provider

- 첫 번째 전략은 ObjectProvider를 사용하는 것!

- LogDemoController.class를 수정

```java
package hello.core.web;

import hello.core.common.MyLogger;
import lombok.RequiredArgsConstructor;
import org.springframework.beans.factory.ObjectProvider;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

import javax.servlet.http.HttpServletRequest;

@Controller
@RequiredArgsConstructor
public class LogDemoController {
    private final LogDemoService logDemoService;
    //myLoggerProvider 를 DI로 받음!
    private final ObjectProvider<MyLogger> myLoggerProvider;

    @RequestMapping("log-demo")
    @ResponseBody
    public String logDemo(HttpServletRequest request){

        //호출될 때 (log-demo 들어왔으면 HTTP Request 생겼으므로
        //MyLogger 의 Lifecycle 내이므로 MyLogger 호출할 수 있다!
        MyLogger myLogger = myLoggerProvider.getObject();
        String requestURL = request.getRequestURI().toString();
        myLogger.setRequestURL(requestURL);

        myLogger.log("controller test");
        logDemoService.logic("testId");
        return "OK";
    }
}
```

- LogDemoService.class를 수정

```java
package hello.core.web;

import hello.core.common.MyLogger;
import lombok.RequiredArgsConstructor;
import org.springframework.beans.factory.ObjectProvider;
import org.springframework.stereotype.Service;

@Service
@RequiredArgsConstructor
public class LogDemoService {

    //Service 도 마찬가지
    private final ObjectProvider<MyLogger> myLoggerProvider;

    public void logic(String id){
        MyLogger myLogger = myLoggerProvider.getObject();
        myLogger.log("service id = " + id);
    }
}
```

- 이후 정상적으로 작동함을 확인할 수 있다

```java
[4ad3218b-2bd1-4d7f-895c-6c70b7f79662] request scope bean create : hello.core.common.MyLogger@6d0f61f0
[4ad3218b-2bd1-4d7f-895c-6c70b7f79662][/log-demo] controller test
[4ad3218b-2bd1-4d7f-895c-6c70b7f79662][/log-demo] service id = testId
[4ad3218b-2bd1-4d7f-895c-6c70b7f79662] request scope bean close : hello.core.common.MyLogger@6d0f61f0
```

- ObjectProvider 덕분에, ObjectProvider.getObject() 호출 시점까지 Request Scope 빈의 생성을 지연할 수 있다.
- ObjectProvider.getObject() 호출 시점에는 HTTP 요청이 진행 중이므로, Request Bean의 생성이 정상 처리된다.
- ObjectProvider.getObject를 LogController, LogService에서 따로 호출해도 같은 HTTP 요청이면 같은 Spring Bean이 반환된다!

---

## 8. 스코프와 프록시

- 프록시 방식을 사용할 수도 있다.
- ../common/MyLogger.class

```java
@Component
@Scope(value= "request", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class MyLogger {
```

- 만약 대상이 클래스면 TARGET_CLASS를,
- 만약 대상이 인터페이스면 INTERFACES를 선택한다.

- 이렇게 함으로써 MyLogger의 가짜 프록시 클래스를 만들어두고, HTTP Request와 상관 없이 가짜 프록시 클래스를 빈에 주입해 둔다.
- LogDemoController.class에서 다음과 같이 Log를 남겨보면

```java
@RequestMapping("log-demo")
    @ResponseBody
    public String logDemo(HttpServletRequest request){

				//Log 찍어보자!
        System.out.println("myLogger = " + myLogger.getClass());

        String requestURL = request.getRequestURI().toString();
        myLogger.setRequestURL(requestURL);
        myLogger.log("controller test");
        logDemoService.logic("testId");
        return "OK";
    }
```

- 다음과 같이 myLogger가 다른 것을 알 수 있다.

```java
//우리가 생성한 MyLogger가 아니다!
myLogger = class hello.core.common.MyLogger$$EnhancerBySpringCGLIB$$c2c98d7d
[26338eed-47de-4161-9293-7d4cbe3231ce] request scope bean create : hello.core.common.MyLogger@378bff63
[26338eed-47de-4161-9293-7d4cbe3231ce][/log-demo] controller test
[26338eed-47de-4161-9293-7d4cbe3231ce][/log-demo] service id = testId
[26338eed-47de-4161-9293-7d4cbe3231ce] request scope bean close : hello.core.common.MyLogger@378bff63
```

- CGLIB이라는 라이브러리로, 내 클래스를 상속받은 가짜 프록시 객체를 만들어 주입한다.

![Spring%20%E1%84%92%E1%85%A2%E1%86%A8%E1%84%89%E1%85%B5%E1%86%B7%20%E1%84%8B%E1%85%AF%E1%86%AB%E1%84%85%E1%85%B5%20%E1%84%87%E1%85%B5%E1%86%AB%20%E1%84%89%E1%85%B3%E1%84%8F%E1%85%A9%E1%84%91%E1%85%B3%201bdbb20920d14453a80afaf30c229dc7/Untitled%204.png](https://github.com/LemonDouble/TIL/blob/main/spring/img/Untitled%2022.png)

- 가짜 프록시 객체는, 요청이 오면 그때 내부에서 진짜 빈을 요청하는 위임 로직이 들어있다.
- 클라이언트가 logic()을 호출하면, 가짜 프록시 메서드를 호출한다.
- 가짜 프록시 객체는 request 스코프의 진짜 myLogger.logic() 을 호출한다.
- 가짜 프록시 객체는 원본 클래스를 상속받아 만들어졌으므로, 이 객체를 사용하는 클라이언트의 입장에서는 원본인지 아닌지도 모르게, 동일하게 사용할 수 있다. (다형성)

- 주의점
    - 마치 싱글톤을 사용하는 것 같지만, 실제로는 다르게 동작하므로 주의해서 사용해야 한다.
    - 이런 특별한 Scope는 꼭 필요한 곳에서만 최소화해서 사용하자. (유지보수 이슈)
