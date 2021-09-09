# Spring MVC 1 : 서블릿, JSP, MVC 패턴

분류: Spring
작성일시: 2021년 9월 7일 오전 9:20

- 먼저 고-대의 방식인 서블릿으로 개발
- 서블릿의 불편한 점이 나오면, 그를 개선해서 JSP로 개발
- JSP의 불편한 점을 MVC 패턴을 적용해서 해결

## 1. 회원 관리 웹 어플리케이션 요구사항

- 회원 정보

  - username
  - age

- 기능

  - 회원 저장
  - 회원 이름 조회

- main/hello/servlet/domain/Member.class

```java

```

- domain/MemberRepository.class

```java

```

- test/.../MemberRepositoryTest.class

```java

```

## 2. 서블릿으로 회원 가입 웹 어플리케이션 만들기

- servlet/web/servlet/MemberFormServlet.class

```java

```

- (같은폴더) MemberSaveServlet.class

```java

```

- (같은폴더) MemberListServlet.class

```java

```

- Servlet을 사용하면 동적으로 HTML을 만들 수 있지만, HTML 작성이 굉장히 불편하다.

```java
//디버깅 할 수 있을까?
w.write("<html>");
        w.write("<head>");
        w.write(" <meta charset=\"UTF-8\">");
        w.write(" <title>Title</title>");
        w.write("</head>");
        w.write("<body>");
        w.write("<a href=\"/index.html\">메인</a>");
        w.write("<table>");
```

- 차라리 HTML 문서에 동적으로 Java 코드를 넣는 방식이 효율적이지 않을까?
  - 그래서 Template Engine이 등장했다!

## 3. JSP로 회원 관리 웹 어플리케이션 만들기

- main/webapp/jsp/members/new-form.jsp

```java

```

- (같은 폴더) save.jsp

```java

```

- (같은 폴더) members.jsp

```java

```

### 서블릿과 JSP의 한계

- 서블릿 개발 시에는, View를 위한 HTML이 Java 코드에 섞여 지저분하고 복잡했다.
- JSP 개발 시에는, Java 코드가 View에 섞여 지저분하고 복잡하다.
- 결과적으로, 서블릿도, JSP도, View와 Logic이 구분되지 못 했다!

- 또한 JSP는 너무 많은 일을 한다.
  - 비즈니스 로직과 View를 모두 담당한다.
  - 이는 유지보수하기 불편함을 야기한다!

## 4. MVC 패턴

- 하나의 서블릿, JSP로 비즈니스 로직, View까지 모두 처리하면 서블릿, 혹은 JSP가 너무 많은 역할을 하게 된다.
- 결과적으로 이는 유지보수의 문제를 야기한다!
- 비즈니스 로직을 수정할때도, HTML 코드를 수정할떄도 같은 JSP 파일을 수정한다고 상상해보자.

- 또한, 비즈니스 로직과 View는 라이프 사이클이 다르다.
  - UI를 일부 수정하는 일과, 비즈니스 로직이 수정되는 일은 각각 다른 사이클로 발생할 가능성이 높다.
  - 또한, 서로에게 영향을 주지 않는다.
  - 대규모 리뉴얼이라면 같이 변하겠지만, 대부분의 경우 버튼 하나 옮기는데 수천줄의 Java Code를 같이 볼 필요는 없다!!

### MVC (Model, View, Controller)의 등장

- Contoller와 View라는 영역으로 서로 역할을 나누자!

  - Controller : HTTP 요청을 받아 파라미터를 검증, 비즈니스 로직을 실행한다. 그리고 View에 전달할 데이터를 조회해 Model에 담는다.
  - Model : View에 출력할 데이터를 담아둔다. 컨트롤러가 Model에 데이터를 담아 View로 전달해 주므로, View는 Model만 알고 있다면 비즈니스 로직, 데이터 접근 방법은 몰라도 된다.
  - View : Model에 담겨있는 데이터를 이용해 화면을 그리는 일에만 집중한다.

- 실제로는 Contoller에서 비즈니스 로직까지 같이 두면, Contoller가 너무 많은 역할을 담당한다. 그래서 일반적으로 비즈니스 로직은 Service라는 계층을 따로 둔다.
  - Contoller : HTTP 요청을 받아 파라미터를 검증, Service 계층을 호출
  - Service : 비즈니스 로직 실행
  - Controller : 로직 결과를 받아 Model에 실음. View에게 Model을 전달

### MVC 패턴 적용

- 서블릿은 컨트롤러로, JSP는 View로 사용
- Mdodel은 HttpServletRequest의 request.setAttribute(), getAttribute() 사용

- /servlet/web/servletmvc/MvcMemberFormListServlet.class

```java

```

- MvcMemberListServlet.class

```java

```

- MvcMemberSaveServlet.class

```java

```

- View 영역
- webapp/WEB-INF/views/new-form.jsp

```java

```

- save-result.jsp

```java

```

- members.jsp

```java

```

---

## 5. 서블릿 기반 MVC의 한계점

- MVC 컨트롤러의 단점

  - 포워드 코드 (dispatcher 얻고, Forward하는 코드) 가 중복된다.
  - Viewpath에서, /WEB-INF/views/, .jsp를 중복적으로 작성하고 있다.
    - 만약 Template Engine이 바뀌거나 폴더 구조가 바뀌면 전부 바꿔줘야 한다!
  - 공통 처리가 어렵다
    - 중복되는 코드를 호출해야 하고, 메소드로 빼더라도 해당 메소드 호출 코드가 중복된다.

- 정리 : 공통 처리가 어렵다.
  - 따라서 컨트롤러 호출 전에 공통 기능을 처리하면 되지 않을까?
  - 수문장 역할(?) 하는 기능 필요. Front Controller 패턴 도입하면 이런 문제를 해결할 수 있다.
