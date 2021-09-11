# Spring MVC 1 : MVC 프레임워크 만들기

분류: Spring
작성일시: 2021년 9월 9일 오후 3:42

## 1. MVC 프레임워크 만들기 - 프론트 컨트롤러 패턴 소개

- 앞에서 서블릿으로 MVC 만들어 봤는데, 공통 처리 코드가 중복되거나 하는 부분이 많았다!
- Front-Controller 패턴을 사용해서 이 문제를 해결해 보자.

- Front-Contoller 패턴 도입 전

  - 공통으로 호출해야 하는 코드를 중복해서 선언/호출해야 했다.

- Front-Contoller 패턴 도입 후

  - Controller에 접근하기 전, Front-Controller를 실행한 후에 Controller를 사용할 수 있다
  - 공통 코드는, Front-Controller에 모아 두면 된다!

- Front-Controller 패턴 특징

  - 프론트 컨트롤러 서블릿이 클라이언트 요청을 받음.
  - 프론트 컨트롤러가 요청에 맞는 Controller 찾아서 호출
  - Front Controller 제외한 나머지 컨트롤러는 서블릿을 사용하지 않아도 됨.

- 스프링 MVC와 프론트 컨트롤러
  - 스프링 MVC의 핵심도 FrontController
  - 스프링 웹 MVC의 DispatcherServlet → FrontController 패턴!

## 2. 프론트 컨트롤러 도입 - v1

![Untitled](Spring%20MVC%201%20MVC%20%E1%84%91%E1%85%B3%E1%84%85%E1%85%A6%E1%84%8B%E1%85%B5%E1%86%B7%E1%84%8B%E1%85%AF%E1%84%8F%E1%85%B3%20%E1%84%86%E1%85%A1%E1%86%AB%E1%84%83%E1%85%B3%E1%86%AF%E1%84%80%E1%85%B5%2067fb40c66c884b7cb5a4d6cd89cd3a9b/Untitled.png)

- 일단 Front Controller 도입해보자!

  - 개선을 한번에 크게 하지 말고, 천천히 개선하자.

- ControllerV1 Interface 구현
  (servlet/web/frontcontroller/v1/ControllerV1.interface)

      ```java
      }
      ```

- Controller 3개 구현

- servlet/web/frontcontroller/v1/controller/MemberFormControllerV1.class

```java

```

- servlet/web/frontcontroller/v1/controller/MemberListControllerV1.class

```java

```

- servlet/web/frontcontroller/v1/controller/MemberSaveControllerV1.class

```java

```

- 그리고, 해당 Controller들 호출할 FrontControllerServletV1 구현한다.

```java

```

## 3. View 분리 (v2)

- 모든 컨트롤러에 View로 이동하는 중복 부분이 있다.

```java

```

- 이 부분을 분리하기 위해, 별도로 View를 처리하는 객체를 만든다.

![Untitled](Spring%20MVC%201%20MVC%20%E1%84%91%E1%85%B3%E1%84%85%E1%85%A6%E1%84%8B%E1%85%B5%E1%86%B7%E1%84%8B%E1%85%AF%E1%84%8F%E1%85%B3%20%E1%84%86%E1%85%A1%E1%86%AB%E1%84%83%E1%85%B3%E1%86%AF%E1%84%80%E1%85%B5%2067fb40c66c884b7cb5a4d6cd89cd3a9b/Untitled%201.png)

- 중복되는 로직을 수행할 MyView를 만든다
- web/frontcontroller/MyView.class

```java

```

- 이후 V1처럼 Contoller Interface를 만든다. Controller는 이제, 직접 Render하지 않고 MyView 객체를 만들어서 반환만 한다.

```java

```

- 이후 각 Controller는 Interface를 구현한다. ( v1과 달라진 점은 Myview 반환 부분 뿐이다.)
- MemberListController.class (나머지도 같음)

```java

```

- 이후, FrontControllerServlet은 반환받은 Myview 호출해 준다.

```java

```

- 반환 타입으로 MyView라는 것을 반환해야 하고, MyView 안에 viewPath가 있으므로, 이후 다른 개발자가 내 코드를 보더라도 viewPath를 넣은 MyView를 반환해야 한다고 알 수 있다!

## 4. Model 추가 (v3)

- Controller 기준으로, Request, Response 필요 없다.

```java

```

- 위의 " request.setAttribute("members", members); " 부분을 보면, Request 객체가 필요해서라기 보다, 단순히 데이터 전달 용으로 Request 객체를 사용했을 뿐이다.
- 따라서, Model 객체를 새로 선언하여 Model을 통해 데이터를 전달하면, > Servlet 기술과 무관하게 Controller는 동작이 가능하다! <
- 이렇게 변환한 경우, 구현 코드 자체도 단순해지고, Test Code 작성도 간편해진다.

---

- 또한, Controller에서 지정하는 View의 이름에 중복이 있다.

```java

```

- 만약 View의 폴더 위치가 변한다면, 저 부분을 전부 일일히 수정해야 한다.
- Contoller는 View의 논리 이름 (new-form, save-result) 만 반환한다.
- 실제 물리 위치는 Front Controller에서 처리하도록 단순화한다.
- 이런 경우, 폴더 위치가 바뀐다면 프론트 컨트롤러만 고치면 된다!

![Untitled](Spring%20MVC%201%20MVC%20%E1%84%91%E1%85%B3%E1%84%85%E1%85%A6%E1%84%8B%E1%85%B5%E1%86%B7%E1%84%8B%E1%85%AF%E1%84%8F%E1%85%B3%20%E1%84%86%E1%85%A1%E1%86%AB%E1%84%83%E1%85%B3%E1%86%AF%E1%84%80%E1%85%B5%2067fb40c66c884b7cb5a4d6cd89cd3a9b/Untitled%202.png)

- Model과 view의 논리 이름을 담을 ModelView 객체 생성

```java

```

- ModelVeiw를 반환하는 Controller 인터페이스 생성

```java

```

- Contoller 인터페이스 받아 실제 구현하는 객체, ModelView 객체를 반환한다.
- MemberSaveControllerV3.class ( 나머지도 비슷하다)

```java

```

- FrontContoller에, viewResolver 호출 / MyView 반환하는 부분 추가해 준다.

```java

```

- Model을 request의 attribute로 매핑하는 render 함수를 overload 해 준다.
- Myview.class

```java

```

## 5. 단순하고 실용적인 컨트롤러

- 앞의 V3 컨트롤러는 서블릿 종속성 제거, View 중복을 제거하는 등 잘 설계된 컨트롤러다.
- 하지만 개발자 입장에서 봤을 때, 항상 ModelView 객체를 생성/반환하는 부분이 귀찮다.
- 좋은 프레임워크는 아키텍쳐 자체도 중요하지만, 실제 개발하는 개발자가 단순하고 편리하게 사용할 수 있어야 한다.

![Untitled](Spring%20MVC%201%20MVC%20%E1%84%91%E1%85%B3%E1%84%85%E1%85%A6%E1%84%8B%E1%85%B5%E1%86%B7%E1%84%8B%E1%85%AF%E1%84%8F%E1%85%B3%20%E1%84%86%E1%85%A1%E1%86%AB%E1%84%83%E1%85%B3%E1%86%AF%E1%84%80%E1%85%B5%2067fb40c66c884b7cb5a4d6cd89cd3a9b/Untitled%203.png)

- Controller는 이제 viewName을 String으로 반환한다.

```java

```

- 이를 통해서 구현이 훨씬 간단해진다!

```java

```

- 구조가 수정된 것을 반영하기 위해, frontcontroller를 수정한다.

```java

```

- 프레임워크와 공통 기능이 수고로워야, 사용하는 개발자가 편리해진다!

## 6. 유연한 컨트롤러 1 - v5

- 한 프로젝트 내에서 어떤 개발자는, ControllerV3 방식으로 개발하고 싶고, 어떤 개발자는 ControllerV4 방식으로 개발하고 싶다면?

- 어댑터 패턴
  - 지금까지 개발한 컨트롤러는 한 가지 방식의 컨트롤러 인터페이스만 사용 가능하다.
  - ControllerV3, V4는 완전히 다른 인터페이스라 호환 불가능하다.
  - 어댑터 패턴을 사용해 프론트 컨트롤러가 다양한 방식의 컨트롤러를 처리할 수 있도록 변경해보자!

![Untitled](Spring%20MVC%201%20MVC%20%E1%84%91%E1%85%B3%E1%84%85%E1%85%A6%E1%84%8B%E1%85%B5%E1%86%B7%E1%84%8B%E1%85%AF%E1%84%8F%E1%85%B3%20%E1%84%86%E1%85%A1%E1%86%AB%E1%84%83%E1%85%B3%E1%86%AF%E1%84%80%E1%85%B5%2067fb40c66c884b7cb5a4d6cd89cd3a9b/Untitled%204.png)

- 핸들러 어댑터를 통 다양한 종류의 컨트롤러가 호출 가능하다!
- 핸들러 : Controller를 더 넓은 범위인 Handler로 변경했다.

  - 이유 : 어댑터가 있으므로, 꼭 컨트롤러 뿐 아니라 어떤 것이든 해당하는 어댑터만 있으면 처리할 수 있기 때문!

- 먼저, MyHandlerAdapter Interface를 선언한다.

```java

```

- ControllerV4HandlerAdapter.class의 경우, String (viewName)을 반환한다.
- 이 때, Adapter의 역할이 명확해진다. Controller는 String을 반환하지만, Adapter는 ModelView를 반환할 책임이 있다. 따라서, ModelView를 만들어서 값을 옮겨담은 후, 반환해 준다.

```java

```

- 이후 FrontControllerServletV5를 구현한다.

  - FrontControllerV5의 로직
    1. 생성할 때, 대응되는 URI (key), Controller(value)를 handlerMappingMap에, 지원되는 Adapter들을 handlerAdapters List에 넣어 초기화한다.
    2. service 메소드 : Servlet에 Request 들어오면, 해당 URI를 받아온다. 이후, handlerMappingMap에서 대응되는 Controller 찾아온다. ( MemberListControllerV3 같은)
    3. 만약 handler가 NULL이라면 (== 못 찾았다면) 404 Error Response 하고 return한다.
    4. 만약 handler가 NULL 아니라면, 해당 Handler에 맞는 Adapter 찾아온다.
    5. 이후, Adapter에 request, response, handler (MemberListControllerV3 같은거) 를 전달한다.
    6. Adapter는 받은 handler를 자신이 담당하는 controller에 맞게 type Casting 한다. 이후 자신의 버전에 맞는 parameter를 생성해, controller에게 넘겨준다. 또한, Parameter로 받은 request 객체에서 Properties를 ParamMap과 같은 DTO에 옮기는 역할 또한 수행한다.
    7. 이후, Controller는 일을 수행하고, 각 버전별로 다른 값을 리턴한다. Adapter의 역할은 return값을 ModelView로 통일하는 역할 또한 한다. 따라서, 만약 Return값이 ModelView가 아니라면, ModelView 객체 생성해서 타입에 맞게 반환한다.
    8. 이후 FrontController는 Adapter로부터 ModelView DTO 반환받는다. FrontController는 ModelView로부터 논리적 이름(new-form 등) 반환받고, 이를 ViewResolver로 전달한다.
    9. ViewResolver는 논리적 이름을 전달받아, Resource Path, 확장자 등을 붙인 뒤, render() function을 가진 MyView 객체를 생성하여 반환한다.
    10. FrontController는 ViewResolver로부터 MyView 객체를 반환받아, 해당 객체의 render() 함수를 호출한다. render 함수를 호출할 때, request, response와 Model 객체를 같이 전달해 준다.
  - FrontControllerV5.class

  ```java

  ```

- 이런 어댑터 방식을 통해, 하위 호환성을 유지하면서도 확장에 유연하도록 만들 수 있다! (OCP)
- Interface 사용해 역할과 구현을 분리함으로써, 유연한 설계를 할 수 있었다.
- 만약 Annotation 이용해 컨트롤러를 더 편하게 만들려면? → Annotation 지원하는 Controller Interface 만들고, Adapter를 추가하면 된다.
- 기능이 추가되더라도, 전체 구조가 변하지 않는다!

## 7. 정리

- 지금까지 한 내용

  - v0 : 그냥 개발
  - v1 : Front-Controller 도입. 기존 구조는 거의 그대로 유지했다.
    TIP : 전체적인 구조를 바꿀 땐, 세세한 부분에 신경쓰지 말고 일단 구조"만" 바꾼다.
    한번에 너무 많은 것을 하지 말자. 작은 변화를 점진적으로 쌓아올려가는 방식으로!
  - v2 : view 분류
    - 단순 반복되는 view 로직을 분리
  - v3 : Model 추가
    - Model을 통해 서블릿 종속성을 제거, Servlet을 나중에 변경하거나 떼어내더라도, 프레임워크는 정상적으로 작동한다.
    - View 이름 중복 제거
  - v4 : 단순하고 실용적 컨트롤러
    - 구조 자체는 v3와 거의 비슷
    - 구현하는 사람 입장에서, ModelView를 직접 생성해 반환하지 않도록, 편리한 인터페이스 제공 (String만 반환해도, FrontController가 ModelView 객체 만들어 처리한다!)
  - v5 : 유연한 컨트롤러
    - 어댑터 도입
    - 어댑터를 추가해, 프레임워크를 유연하고 확장성 있게 설계!

- Spring MVC도 같은 구조 가지고 있다!

  - @RequestMapping("/hello");
  - RequestMappingHandlerAdapter가 위 Annotation 처리한다.

  - 유연한 구조를 통해, Java에 갑자기 Annotation이 추가되었을 때도 간단히 Annotation을 지원할 수 있었다.
