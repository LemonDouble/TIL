# Spring MVC 1 : 서블릿

분류: Spring
작성일시: 2021년 9월 2일 오전 11:26

## 1. Hello Servlet

- java../basic/HlloServlet.java

```java

```

- [application.properties](http://application.properties) ( HTTP 요청을 Debug 형태로 볼 수 있다)

```java
logging.level.org.apache.coyote.http11=debug
```

- 웹 어플리케이션 서버의 요청/응답 구조

![Untitled]()

## 2. HttpServletRequest 개요

- HTTP Message를 직접 파싱하는 것은 불편!
- 따라서, 서블릿이 HTTP를 사용하기 쉽도록 대신 파싱한다.
- 이후, 그 결과를 HttpServletRequest 객체에 담아서 제공

- 그 이외에 추가 기능도 같이 제공한다!

  - HTTP 요청 시작부터 끝날 때까지 유지되는 임시 저장소 기능

    - 저장 : request.setAttribute(name, value)
    - 조회 : request.getAttribute(name)

  - 세션 관리 기능
    - request.getSession(create:true)

## 3. HttpServletRequest 기본 사용법

- java.../servlet/basic/request/RequestHeaderServlet.class

```java

```

- Host를 편리하게 조회하거나, Accept-Language, Cookie, Content 등도 편하게 조회할 수 있다

## 4. Http 요청 데이터 - 개요

- 주로 3가지 방법 사용
  - GET : 검색, 필터, 페이징등에서 많이 사용 | URL Query Parameter로 데이터 전달
  - POST : 회원 가입, 상품 주문 등.. | context-type : application/x-www-form-urlencoded
  - HTTP message Body에 데이터를 직접 담아서 요청
    - HTTP API에서 주로 사용
    - JSON, XML, TEXT
    - POST, PUT, PATCH

## 5. Http 요청 데이터 - GET 쿼리 파라미터

- [http://example.com](http://example.com)/someURI?username=hello&age=20
  - ?를 시작으로 Query Parameter 보낼 수 있다.
  - 추가 파라미터는 &로 구분한다.

```java

```

- http://localhost:8080/request-param?username=hello&username=hello2&age=20
  - 위처럼 username이라는 파라미터가 중복일 수도 있다.
  - 이런 경우, request.getParameter() 사용시 첫 번째 값만 나온다.
  - 하지만 보통 이렇게 중복으로 보내는 경우는 거의 없다..

## 6. Http 요청 데이터 - POST HTML Form

- content-type이 application/x-www-form-urlencode
- 메세지 바디에 들어오는 파라미터는 GET과 같다. (username=hello&age=20)

- 하지만 GET과 받는 방식은 똑같다!

  - request.getParameter()로 get 방식도, post 방식도 처리할 수 있다!

- 클라이언트(브라우저) 입장에서는, 두 방식에 차이가 있지만, 서버 입장에서는 둘의 형식이 동일하므로 같은 메소드로 조회할 수 있다.

## 7. Http 요청 데이터 - API 메시지 바디 - 단순 텍스트

- HTTP Message Body에 데이터를 직접 담아서 요청하는 경우

  - 주로 JSON 사용. TEXT, XML등도 사용가능
  - POST, PUT, PATCH

- request/RequestBodyStringServlet.class

```java

```

## 8. Http 요청 데이터 - API 메시지 바디 - JSON

- POST [http://localhost:8080/request-body-json](http://localhost:8080/request-body-json)
- content-type: application/json
- message body: {"username": "hello", "age": 20}

```java

```

- ObjectMapper 클래스는 Jackson이라는 JSON 변환 라이브러리가 제공하는 클래스다.
  - Spring Boot로 Spring MVC 선택하면, 기본으로 Jackson 라이브러리를 제공!

## 9. HttpServletResponse : 기본 사용법

- HTTP 응답 메세지를 생성!

  - HTTP 응답코드 지정
  - 헤더 생성
  - 바디 생성

- 편의 기능 제공

  - Content-Type, 쿠키, Redirect 등

- basic.response.ResponseHeaderServlet.class

```java

```

- 이후 크롬으로 들어가 F12 → Network에서 확인 가능하다.

![Untitled](Spring%20MVC%201%20%E1%84%89%E1%85%A5%E1%84%87%E1%85%B3%E1%86%AF%E1%84%85%E1%85%B5%E1%86%BA%20c9f963c5e7b64fa18eb5e5b5affe0891/Untitled%201.png)

## 10. HttpServletResponse : 응답 데이터 - 단순 텍스트,HTML

- 단순 텍스트

  - response.getWriter().println("텍스트");

- HTML

  - response/ResponseHtmlServlet.class

  ```java

  ```

## 11. HttpServletResponse : 응답 데이터 - API, JSON

- response/ResponseJsonServlet.class

```java

```
