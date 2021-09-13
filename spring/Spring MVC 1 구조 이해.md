# Spring MVC 1 : 구조 이해

분류: Spring
작성일시: 2021년 9월 12일 오후 2:58

## 1. 스프링 MVC 전체 구조

![Untitled](Spring%20MVC%201%20%E1%84%80%E1%85%AE%E1%84%8C%E1%85%A9%20%E1%84%8B%E1%85%B5%E1%84%92%E1%85%A2%20af03deadbbb54442a998ffc25ab3eaf7/Untitled.png)

- 이름을 제외하곤 거의 똑같다!
  - FrontController = DispatcherServlet
  - handlerMappingMap = HandlerMapping
  - MyHandlerAdapter = HandlerAdapter
  - ModelView = ModelAndView
  - viewResolver(Method) = ViewResolver(Interface)
  - MyView = View(interface)

### DispatcherServlet 구조 살펴보기

- Spring MVC도 Front Controller 패턴으로 구현되어 있다.
  - Spring MVC의 Front Controller가 DispatcherServlet이다.
  - DispatcherServlet이 Spring MVC의 핵심!

![Untitled](Spring%20MVC%201%20%E1%84%80%E1%85%AE%E1%84%8C%E1%85%A9%20%E1%84%8B%E1%85%B5%E1%84%92%E1%85%A2%20af03deadbbb54442a998ffc25ab3eaf7/Untitled%201.png)

- DispatcherServlet도 부모 클래스에서 HttpServlet을 상속 받아 사용하고, 서블릿으로 동작한다. (DispatcherServlet- > FrameworkServlet → HttpServletBean → HttpServlet)
- Spring Boot는 DispatcherServlet을 서블릿으로 자동 등록하며, 모든 경로(urlPatterns="/") 에 대해 매핑한다.

  - 이때, 더 자세한 경로가 우선순위가 높다.
  - 따라서 기존에 내가 등록한 서블릿도 함께 동작한다.

- 요청 흐름

  - 서블릿 호출 → HttpServlet이 제공하는 service()가 호출된다.
  - Spring MVC는 DispatcherServlet의 부모인 FrameworkServlet에서 service()를 오버라이드 해 두었다.

  (FramworkServlet.class의 오버라이드된 service)

  ```java

  ```

  - FrameworkServlet.service()를 시작으로 여러 메서드가 호출되면서, DispatcherServlet.doDispatch()가 호출된다 (중요!)

  ```java

  ```

  - 동작 순서

    1. 핸들러 조회 : 핸들러 매핑을 통해, 요청 URL에 매핑된 핸들러를 조회한다.
    2. 핸들러 어댑터 조회 : 핸들러를 실행할 수 있는 핸들러 어댑터를 조회한다.
    3. 핸들러 어댑터 실행 : 핸들러 어댑터를 실행한다.
    4. 핸들러 실행 : 핸들러 어댑터가 실제 핸들러를 실행한다.
    5. ModelAndView 반환 : 핸들러 어댑터는 핸들러가 반환하는 정보를 ModelAnadView로 변환해서 반환한다.
    6. viewResolver 호출 : 뷰 리졸버를 찾고 실행한다.
    7. View 반환 : 뷰 리졸버는 뷰의 논리 일므을 물리 이름으로 바꾸고, 렌더링 역할을 담당하는 뷰 객체를 반환한다.
    8. 뷰 렌더링 : 뷰를 통해 뷰를 렌더링한다.

  - Spring MVC의 큰 강점은, DispatcherServlet 코드 변경 없이 원하는 기능을 변경하거나, 확장할 수 있다.

    - 대부분의 기능을 확장 가능하게 인터페이스로 제공한다!
    - 제공하는 Interface만 구현해서, DispatchServlet에 등록하면, 직접 컨트롤러를 만들 수도 잇다.

  - 주요 Interface 목록
    - 핸들러 매핑 : org.springframework.web.servlet.HandlerMapping
    - 핸들러 어댑터 : org.springframework.web.servlet.HandlerAdapter
    - 뷰 리졸버 : org.springframework.web.servlet.ViewResolver
    - 뷰 : org.springframework.web.servlet.View

### 정리

- 사실 99.99%의 기능들은 지난 10년동안 Spring이 구현해 놓은 것이 많다.
- 따라서, 직접 기능확장/컨트롤러를 구현하는 일은 거의 없다.
- 하지만 핵심 동작 원리를 알아두어야, 향후 문제가 발생했을 때 Fix가 가능하다.
- 또한, 확장 포인트가 필요할 때 어느 부분을 확장해야 할지 감을 잡을 수 있다.

## 2. 핸들러 매핑과 핸들러 어댑터

- 지금은 Annotation 기반으로 사용하므로 거의 사용하지 않지만, 과거에는 주로 사용했던 방식이다.
- 과거 제공했던 Controller

```java

```

- 참고 : Controller 인터페이스는 @Controller 어노테이션과 완전히 다르다!

- 이 Controller Interface를 구현해 만들어 보자.
- java/hello/servlet/web/springmvc/old/OldController.class

```java

```

- 이후, [http://localhost:8080/springmvc/old-controller](http://localhost:8080/springmvc/old-controller) 로 들어가면 sout 찍혀있는거 확인할 수 있다.
- 이 Controller는 어떻게 호출될 수 있었을까?

- 이 Controller 호출 위해서는 다음 2가지가 필요하다.

  - HandlerMapping
    - 핸들러 매핑에서 이 컨트롤러를 찾을 수 있어야 한다.
    - 예 ) 스프링 빈의 이름으로 핸들러를 찾을 수 있는 핸들러 매핑이 필요하다.
  - HandlerAdapter
    - 핸들러 매핑을 통해 찾은 핸들러를 실행할 수 있는 핸들러 어댑터가 필요하다.
    - 예 ) Controller 인터페이스를 실행할 수 있는 핸들러 어댑터를 찾고 실행해야 한다.

- Spring은 이미 필요한 Handler Mapping, Adapter를 구현해 뒀다!
  - 실제로는 더 많지만, 중요한 부분 위주로 설명하기 위해 일부가 생략되
  - HandlerMapping
    - 0순위 = RequestMappingHandlerMapping : Annotation 기반 컨트롤러인 @RequestMapping에서 사용
    - 1순위 = BeanNameUrlHandlerMapping : Spring Bean의 이름으로 핸들러를 찾는다.
      - 위 Controller는 이 HanderMapping으로 찾음!
  - HandlerAdapter
    - 0순위 = RequestMappingHandler : Annotation 기반 컨트롤러인 @RequestMapping에서 사용
    - 1순위 = HttpRequestHandlerAdapter : HttpRequestHandler (interface) 처리
    - 2순위 = SimpleControllerHandlerAdapter : Controller 인터페이스 ( Annotation X)
      - 위 Controller는 이 Adapter로 찾음!

![Untitled](Spring%20MVC%201%20%E1%84%80%E1%85%AE%E1%84%8C%E1%85%A9%20%E1%84%8B%E1%85%B5%E1%84%92%E1%85%A2%20af03deadbbb54442a998ffc25ab3eaf7/Untitled%202.png)

- 따라서, 위 내용을 정리하면
  1. Handler Mapping으로 핸들러 조회
     - HandlerMapping 순차 실행해서 핸들러를 찾는다.
     - 우리가 작성한 Controller 경우, Bean 이름으로 핸들러를 찾는 BeanNameUrlHandlerMapping가 실행에 성공, 핸들러인 OldController 반환한다.
  2. Handler Adapter 조회
     - Handler Adapter의 supports()를 순서대로 호출한다.
     - SimpleControllerHandlerAdapter 가 Controller interface 지원하므로 대상이 된다.
  3. Handler Adapter 실행
     - DispatcherServlet이, 조회한 SimpleControllerHandlerAdapter 실행하며 Handler 정보도 같이 넘겨준다.
     - SimpleControllerHandlerAdapter 는, handler인 OldController를 내부에서 실행, 그 결과를 반환한다.

---

- 하지만 하나만 가지곤 잘 모르겠다..
- 다른것도 한번 만들어 보자

- HttpRequestHandler
- 위와 같은 폴더 (old/MyHttpRequestHandler.class)

```java

```

- 마찬가지로 [http://localhost:8080/springmvc/request-handler](http://localhost:8080/springmvc/request-handler) 접속하면 sout 나온다.

1. BeanNameUrlHandlerMapping 로 MyHttpRequestHandler 얻음
2. HttpRequestHandlerAdapter가 HttpRequestHandler Support하므로 반환됨
3. HttpRequestHandlerAdapter가 MyHttpRequestHandler를 실행!

### 3. 뷰 리졸버 (View Resolver)

- 위에서 만든 OldController가 ModelAndView 반환하도록 한다.

```java

```

- [http://localhost:8080/springmvc/old-controller](http://localhost:8080/springmvc/old-controller) 로 접속하면 접속이 안 된다!
- resources/application.properties에 다음 두 줄 추가해준다.

```java
spring.mvc.view.prefix=/WEB-INF/views/
spring.mvc.view.suffix=.jsp
```

- 이후 재접속하면 제대로 접속된다.

---

- InternalResourceViewResolver

  - Spring Boot는 InternalResourceViewResolver라는 View Resolver를 자동으로 등록한다.
  - 이 때, application.properties에 등록한 spring.mvc.view.prefix, suffix를 사용해 등록한다.

- Spring Boot가 자동 등록하는 View Resolver (실제론 더 많지만, 일부 생략)

  1. BeanNameViewResolver: Bean 이름으로 뷰를 찾아 반환한다.
     (예시 : 엑셀 파일용 View Resolver 개발 등에 사용)
  2. InternalResourceViewResolver : JSP를 처리할 수 있는 View를 반환한다.

- View Resolver의 작동 방법

  1. Handler Adapter 호출
     - Handler Adapter를 통해 new-form이라는 논리 뷰 이름을 획득한다.
  2. ViewResolver 호출
     - new-form이라는 뷰 이름으로 viewResolver를 순서대로 호출한다.
     - BeanNameViewResolver 는 Bean 이름으로 찾으니, new-form이라는 Spring Bean으로 등록된 view를 찾아야 하나 없으므로 실패한다.
     - InternalResourceViewResolver가 호출되고, 이 Resolver는 InternalResourceView를 반환한다.
  3. InternalResourceView의 경우, JSP처럼 forward()를 호출해 처리할 수 있는 경우 사용한다.
  4. view.render()가 호출되고, InternalResourceView는 forward() 사용해 JSP 실행한다.

- TIP
  - InternalResourceViewResolver는, JSTL 있으면 InternalResourceView 상속받은 JstlView 반환한다. (약간의 부가 기능 추가)
  - 다른 View는 실제 View를 렌더링하지만, JSP의 경우 forward() 통해 해당 JSP로 이동(실행) 해야만 렌더링이 된다. JSP 제외하곤 나머지는 forward() 없이 바로 렌더링된다.
  - Thymeleaf View Template 사용하려면 ThymeleafViewResolver 등록해야 한다. 최근에는 라이브러리만 추가하면 Spring Boot가 이런 작업도 자동화 해 준다!

---

## 4. Spring MVC - 시작하기

- 현재 Spring이 제공하는 Controller는 Annotation 기반으로, 매우 유용/실용적이다.

- @RequestMapping

  - 이전에는 Spring MVC 부분이 약해서, 스트럿츠 등 외부 프레임워크를 사용헀었음.
  - 하지만 @RequestMapping 기반의 Annotation Controller 등장하며 MVC 부분도 Spring의 완승으로 끝났다.

- @RequestMapping

  - RequestMappingHandlerMapping
  - RequestMappingHandlerAdapter
  - 이것이 지금의 Spring에서 주로 사용하는 Annotation 기반의 핸들러 매핑 및 어댑터
  - 99.9%는 이 방식의 컨트롤러의 사용!

- @RequestMapping 기반 컨트롤러 - v1
- springmvc/v1/SpringMemberFormControllerV1.class

```java

```

- Controller :
  - Spring이 자동으로 Spring Bean으로 등록 (Controller 내부에 @Component 있어, Component Scan 대상이 됨)
  - Spring MVC에서 Annotation 기반 컨트롤러로 인식한다.
- RequestMapping
  - 요청 정보를 Mapping, 해당 URL이 노출되면 이 메서드가 호출된다.
  - Annotation을 기반으로 동작하므로, 메서드 이름은 임의로 지으면 된다.
- ModelAndView
  - Model과 View 정보를 담아 반환하면 된다.

---

- RequestMappingHandlerMapping는 @RequestMapping, @Controller가 클래스 레벨에 있는 경우 매핑 정보로 인식한다.
- 따라서, 다음 코드도 동일하게 동작한다.

```java

```

---

### V1 : Annotation 기반 Controller

- SpringMemberFormControllerV1.class

```java

```

- SpringMemberSaveControllerV1.class

```java

```

- SpringMemberListControllerV1.class

```java

```

## 5. Spring MVC - 컨트롤러 통합

- @RequestMapping을 보았을 때, Class 단위가 아니라 메서드 단위로 적용되는 것을 확인할 수 있다. 따라서, 컨트롤러 클래스를 유연하게 하나로 통합할 수 있다!

- v2/SpringMemberControllerV2.class

```java

```

- 같은 Controller를 통합한건 좋지만, ModelAndView 만들어 반환하니까 불편한 부분이 있다!

## 6. Spring MVC - 실용적인 방식

- 실무에서 가장 많이 사용하는 방식!

- v3/SpringMemberControllerV3.class

```java

```

- viewName을 String으로 반환 가능하다.
- @RequestParm : HTTP 요청 파라미터를 @RequestParam으로 받을 수 있다.
  - @RequestParam("username")은, request.getParameter ("username")과 거의 같은 코드이다.
- @RequestMapping → @GetMapping, @PostMapping 구분
  - 기능에 맞게 HTTP Method를 제한함으로써 더 좋은 설계를 할 수 있다!
