# Spring MVC 1 : 웹 애플리케이션의 이해

분류: Spring
작성일시: 2021년 7월 21일 오후 8:02

## 1. 웹 서버, 웹 어플리케이션 서버

- 웹 서버 (Web Server)

  - HTTP 기반으로 동작
  - 정적 리소스 제공, 기타 부가기능 제공
  - 정적(파일) HTML, CSS, JS, 이미지, 영상
  - 예) NGINX, APACHE

- 웹 어플리케이션 서버 (WAS : Web Application Server)

  - HTTP 기반으로 동작
  - Web Server 기능 제공
  - - 프로그램 코드를 실행해 어플리케이션 로직 수행
      - 동적 HTML, HTTP API(JSON)
      - 서블릿, JSP, 스프링 MVC
  - 예) 톰캣(Tomcat) , Jetty, Undertow

- 웹 서버와 WAS의 차이

  - 웹 서버는 정적 리소스(파일), WAS는 어플리케이션 로직
  - 하지만 실제로는 둘의 용어도, 경계도 모호함
    - 웹서버도 프로그램을 실행하는 기능을 포함하기도 함
    - 웹 어플리케이션 서버도 웹 서버의 기능을 제공함
  - JAVA는 서블릿 컨테이너 기능을 제공하면 WAS
    - 서블릿 없으 Java Code를 실행하는 서버 프레임워크도 있음
  - WAS는 어플리케이션 코드 실행하는데 더 특화

- 웹 시스템 구성 - WAS, DB

  - Client → WAS (Application 로직, 정적 리소스 처리) → DB

  - WAS, DB만으로도 시스템 구성 가능
  - WAS는 정적 리소스, 어플리케이션 로직 전부 제공 가능

  - 하지만 이런 경우, WAS가 너무 많은 역할을 담당, 서버 과부하 우려
  - 가장 비싼 어플리케이션 로직이 정적 리소스 때문에 수행이 어려울 수 있음
    - 어플리케이션 로직 : 주문 처리 등등..
    - 정적 리소스 (이미지 전송 등..) 으로 어플리케이션 로직이 실행되지 않을 수 있음
  - WAS 장애시, 오류 화면도 노출 불가능

- 웹 시스템 구성 - WEB Server, WAS, DB

  - Clinet → Web Server (정적 리소스 처리 ) → WAS (어플리케이션 로직) → DB

  - 정적 리소스는 웹 서버가 처리
  - 웹 서버는 어플리케이션 로직같은 동적인 처리가 필요하면 WAS에 요청을 위임
  - WAS는 주요한 어플리케이션 로직 처리만 전담

![Untitled](https://github.com/LemonDouble/TIL/blob/main/spring/img/Untitled%2029.png)

## 2. 서블릿

![Untitled](https://github.com/LemonDouble/TIL/blob/main/spring/img/Untitled%2030.png)

- HTTP 요청은 단순한 텍스트, 처음부터 끝까지 구현하려면 소켓 연결부터 엄청나게 많은 업무를 직저 해 줘야 한다.
- 하지만 이걸 일일히 구현하는건 비효율적!

- 서블릿은 핵심 비즈니스 로직 작성을 제외한 부분들을 서블릿이 처리해 준다!

- 서블릿의 모습

```java
// /hello URL로 들어오는 요청을 helloServlet이 처리해 준다
@WebServlet(name = "helloServlet", urlPatterns = "/hello")
public class HelloServlet extends HttpServlet {
 @Override
 protected void service(HttpServletRequest request, HttpServletResponse response){
 //애플리케이션 로직
 }
}
```

- Http Request를 편리하게 처리할 수 있는 HttpServletRequest
- Http Response를 편리하게 제공할 수 있는 HttpServletResponse

![Untitled](https://github.com/LemonDouble/TIL/blob/main/spring/img/Untitled%2031.png)

1. 브라우저가 /hello로 요청
2. Http 요청 메세지를 기반으로 request, response 객체를 만듬
3. helloServlet을 실행시킴
4. helloServlet이 끝나고 return되면, response에 작성한 내용을 바탕으로 response해줌.

![Untitled](https://github.com/LemonDouble/TIL/blob/main/spring/img/Untitled%2032.png)

- 서블릿 컨테이너
  - Tomcat처럼 서블릿을 지원하는 WAS를 서블릿 컨테이너라고 함.
  - 서블릿 컨테이너는 서블릿 객체를 생성,초기화,호출,종료하는 Lifecycle 관리를 해 줌.
  - 서블릿 객체는 >> 싱글톤 << 으로 관리
    - 고객의 요청이 올 때마다 객체를 생성하는 것은 비효율
    - 최초 로딩 시점에 서블릿 객체를 미리 만들어두고 재활용
    - 모든 고객 요청은 동일한 서블릿 객체 인스턴스에 접근
    - 따라서, 공유 변수 사용 주의!
    - 서블릿 컨테이너 종료시 함꼐 종료
  - JSP도 서블릿으로 변환되어 사용
  - 동시 요청을 위한 멀티쓰레드 처리 지원

## 3. 동시 요청 - 멀티 쓰레드

- 쓰레드
  - 어플리케이션 코드를 하나하나 순차적으로 실행하는 것은 쓰레드
  - JAVA 메인 메서드를 처음 실행하면, main이라는 이름의 쓰레드가 실행
  - 쓰레드가 없다면, Java Application 실행이 불가능
  - 쓰레드는 한 번에 하나의 코드 라인만 수행
  - 동시 처리가 필요하면, 쓰레드를 추가로 생성

---

- 첫번째 방법 : 쓰레드 하나 사용

![Untitled](https://github.com/LemonDouble/TIL/blob/main/spring/img/Untitled%2033.png)

- 문제 : 만약 한 요청이 처리가 지연되면, 요청 전체가 뻗는다. (비동기 가능하지만 동기라고 치고..)

---

- 두번째 방법 : 요청이 올 때마다 새로 쓰레드 생성

![Untitled](https://github.com/LemonDouble/TIL/blob/main/spring/img/Untitled%2034.png)

- 장점

  - 동시 요청 처리 가능
  - 리소스( CPU, 메모리 ) 가 허용할 때 까지 처리 가능
  - 하나의 쓰레드가 지연되더라도, 나머지 쓰레드는 정상 동작한다.

- 단점
  - 쓰레드 생성 비용은 매우 비싸다.
    - 고객의 요청이 올 때마다 쓰레드를 생성하면, 응답 속도가 늦어진다.
  - 쓰레드는 Context Switching 비용이 발생한다.
  - 쓰레드 생성에 제한이 없다.
    - 고객 요청이 너무 많다면, CPU, 메모리 임계점을 넘어 서버가 죽을 수 있다.

---

- 세 번째 방법 : Thread Pool

![Untitled](https://github.com/LemonDouble/TIL/blob/main/spring/img/Untitled%2035.png)

- 풀 안에 쓰레드를 미리 만들어 놓고, 사용 후 반납한다.

- 특징

  - 필요한 쓰레드를 쓰레드 풀에 보관/관리한다.
  - 쓰레드 풀에 생성 가능한 쓰레드의 최대치를 관리한다.
  - Tomcat은 최대 200개 기본 설정 (변경 가능)

- 사용

  - 쓰레드가 필요하면, 이미 생성되어 있는 쓰레드를 쓰레드 풀에서 꺼내서 사용한다.
  - 사용을 종료하면 쓰레드 풀에 해당 쓰레드를 반납한다.
  - 최대 쓰레드가 모두 사용중이어서 쓰레드 풀에 쓰레드가 없다면?
    - 기다리는 요청을 거절할 수 있다.
    - 또는, 특정 숫자만큼만 대기하도록 설정할 수 있다.

- 장점

  - 쓰레드가 미리 생성되어 있으므로, 쓰레드 생성/종료 비용 (CPU) 가 절약되고 응답 시간이 빠르다.
  - 생성 가능한 쓰레드의 최대치가 있으므로, 너무 많은 요청이 들어와도 기존 요청은 안전하게 처리할 수 있다.

- TIP!

  - WAS의 주요 튜닝 포인트는 최대 쓰레드 (Max Threa) 수이다.
  - 최대 쓰레드 너무 낮다면?
    - 동시 요청이 많으면, 서버 리소스는 여유롭지만 클라이언트는 응답 지연
  - 너무 높다면?

    - 동시 요청이 많으면, CPU, 메모리 리소스 임계점 초과로 서버 다운

  - 장애 발생시 대처법
    - 클라우드 : 일단 서버부터 늘리고, 이후에 튜닝
    - 로컬 : 일단 열심히 튜닝..

- 쓰레드 풀의 적정 숫자
  - 적정 숫자는 어떻게 찾나요?
  - 로직의 복잡도, CPU, 메모리, IO 리소스 상황에 따라 모두 다름
  - 성능 테스트 하자
    - 최대한 실제 서비스와 유사하게 테스트를 시도
    - 툴 : 아파치 AB, 제이미터 (JMeter), nGrinder

---

- WAS의 멀티 쓰레드 지원 : 핵심
  - 멀티 쓰레드에 대한 부분은 WAS가 처리
  - 개발자가 멀티 쓰레드 관련 코드를 신경쓰지 않아도 됨.
  - 개발자는 마치 싱글 쓰레드 프로그램을 만들듯 편리하게 소스 코드를 개발
  - 하지만 멀티 쓰레드 환경이므로 싱글톤 객체 (서블릿, Spring Bean)은 주의해서 사용

## 4. 자바 백엔드 웹 기술의 역사

과거

- 서블릿 - 1997 : HTML 동적 생성이 어려움..
- JSP : HTML 생성은 편리하지만, VIEW부터 비즈니스 로직까지 너무 많은 역할을 담당
- 서블릿 + JSP, MVC 패턴 사용 : Model, View, Controller 역할 나누어 개발
- 여러 MVC 프레임워크 등장 : 스트럿츠, 웹워크, 스프링 MVC (과거 버전)

현재

- 애노테이션 기반의 스프링 MVC 등장
  - @Controller
  - MVC 프레임워크 춘추 전국 시대 마무리
- 스프링 부트의 등장
  - 스프링 부트는 서버를 내장
  - 과거에는 서버에 WAS 직접 설치하고, 소스는 War 파일 만들어 설치한 WAS에 배포
  - 스프링 부트는 빌드 결과(Jar)에 WAS 서버 포함 → 빌드 배포 단순화

최신 기술

- Web Servlet - Spring MVC
- Web Reactive - Spring WebFlux

  - 특징

    - 비동기 Non-Blocking 처리
    - 최소 쓰레드로 최대 성능 - 쓰레드 Context Switching 비용 효율화
    - 함수형 스타일 개발 - 동시처리 코드 효율화
    - 서블릿 기술 사용 X

  - 그러나
    - WebFlux는 기술적 난이도 매우 높음
    - 아직 RDB 지원 부족
    - 일반 MVC의 쓰레드 모델도 충분히 빠르다.
    - 아직 실무에서 많이 사용하진 않음.
