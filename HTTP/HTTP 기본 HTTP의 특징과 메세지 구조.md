# HTTP 기본 : HTTP의 특징과 메세지 구조

분류: HTTP
작성일시: 2021년 7월 19일 오후 11:04

## 1. HTTP 개론

- HTTP : HyperText Transfer Protocol
- HyperText : 하이퍼링크(참조) 를 통해 독자가 한 문서에서 다른 문서로 즉시 접근할 수 있는 텍스트

- HTML, TEXT, Image, 음성, 영상, 파일, JSON, XML(API) 등 거의 모든 형태의 데이터를 전송 가능하다.
- 서버간 데이터를 주고 받을떄도 대부분 HTTP 사용

- HTTP 1.1 (RFC 7230~7235)
    - 1997년 나옴, 가장 많이 사용되는 버전!
    - 사용하는 거의 모든 기능이 들어있음
    - HTTP 2, 3 등도 있지만, 기능 추가보다는 성능 개선에 초점에 맞추어져 있다.

- 기반 프로토콜
    - HTTP 1.1, HTTP 2 : TCP 위에서 동작
    - HTTP 3 : UDP 위에서 동작

---

- Protocol을 직접 확인해보자!
    - Chrome 개발자 도구 열기
    - Network 창 열기
    - 아래 표에서, 우클릭해서 Protocoil 활성화
    - google → HTTP 3,  Naver → HTTP 2, 1.1 등 사용

## 2. HTTP의 특징 : 클라이언트-서버 구조

- Request / Response 구조
    - 클라이언트는 서버에 요청을 보내고, 응답을 대기
    - 서버가 요청에 대한 결과를 만들어서 응답

- Client / Server 를 분리!
- Server : 비즈니스 로직, 데이터 등을 처리하는데 집중
- Client : UI 생성, 사용성 개선 등에 집중

## 3. HTTP의 특징 : Stateless

- 무상태 프로토콜 (Stateless)
    - 서버가 클라이언트의 상태를 보존하지 않음
    - 장점 : 서버 확장성 높음 (Scale Out : 수평 확장 유리)
    - 단점 : 클라이언트가 추가 데이터 전송이 필요함

- Stateful
    - 중간에 다른 서버로 바뀌면 장애가 발생한다.
    - 중간에 다른 서버로 바뀐다면, State 정보를 다른 서버에 이전해줘야 하낟.

- StateLess
    - 중간에 다른 서버로 바뀌어도 문제가 없다.
    - 응답 서버를 쉽게 바꿀 수 있으므로 무한한 서버 증설이 가능하다!

- Stateless의 한계
    - 모든 것을 무상태로 설계할 수 없는 경우도 있다.
    - 로그인이 필요없는 단순한 서비스 소개 화면 → OK
    - 로그인이 있는 경우 → X!

    - 일반적으로 브라우저 쿠키나, 서버 세션등을 사용해 상태 유지
    - State는 최소한으로 사용

## 4. HTTP의 특징 : 비연결성 (ConnectionLess)

- Connection을 유지하는 모델의 문제점
    - 서버가 당장 데이터를 전송하지 않더라도, Connection을 유지해야 한다.
    - Connection을 유지하는데 서버의 자원이 요구된다.

- ConnectionLess
    - HTML은 기본이 연결을 유지하지 않는 모델
    - 일반적으로 초 단위 이하의 빠른 속도로 응답
    - 1시간동안, 수천명이 서비스를 사용해도 실제 서버에서 동시에 처리하는 요청은 수십개 이하로 매우 작다.
    - 서버 자원을 매우 효율적으로 사용할 수 있다.
    - 하지만 TCP/IP 연결을 새로 맺어야 함 → RTT 시간만큼 더 걸림
    - 지금은 HTTP 지속 연결 (Persistent Connection으로 문제 해결 : HTTP 1.1부터)
    - HTTP/2, HTTP/3에서 더 많이 최적화

- Stateless를 기억하자!
    - 가장 어려운 업무 : 같은 시간에 딱 맞추어 발생하는 대용량 트래픽

        (수강신청, 명절 KTX 예약 등..)

    - 최대한 Stateless하게..
        - 첫 Page는 Login 없는 정적 Page 하나 둠
        - 그 페이지를 거쳐 참가 Page를 누르도록 하거나..

## 5. HTTP 메세지

![HTTP%20%E1%84%80%E1%85%B5%E1%84%87%E1%85%A9%E1%86%AB%20HTTP%E1%84%8B%E1%85%B4%20%E1%84%90%E1%85%B3%E1%86%A8%E1%84%8C%E1%85%B5%E1%86%BC%E1%84%80%E1%85%AA%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A6%E1%84%8C%E1%85%B5%20%E1%84%80%E1%85%AE%E1%84%8C%E1%85%A9%2097f6c6f126994738a6df471389531550/Untitled.png](HTTP%20%E1%84%80%E1%85%B5%E1%84%87%E1%85%A9%E1%86%AB%20HTTP%E1%84%8B%E1%85%B4%20%E1%84%90%E1%85%B3%E1%86%A8%E1%84%8C%E1%85%B5%E1%86%BC%E1%84%80%E1%85%AA%20%E1%84%86%E1%85%A6%E1%84%89%E1%85%A6%E1%84%8C%E1%85%B5%20%E1%84%80%E1%85%AE%E1%84%8C%E1%85%A9%2097f6c6f126994738a6df471389531550/Untitled.png)

- 공백 (CRLF) 는 생략 불가능! 무조건 있어야 한다.
- message body 없다면 생략 가능하다.

- 시작 라인 (Start-line)
    - start-line = request-line or status-line
    - request-line = method SP(공백) request-target SP HTTP-version CRLF(엔터)
        - 예시 : GET /search?q=hello&hl=ko HTTP/1.1
        - method : GET, POST, PUT, DELETE .. → 서버가 수행해야 하는 동작 지정
        - request-target : 보통 "/"로 시작하는 절대 경로로 시작.
        - HTTP-version : HTTP/1.1...

    - status-line = HTTP-Version SP status-code SP reason-phrase CRLF
        - 예시 : HTTP /1.1 200 OK
        - HTTP-version : HTTP/1.1....
        - status-code : 200 : 성공, 400 : 클라이언트 요청 오류 ...
        - reason-phrase : 사람이 이해할 수 있는 짧은 코드 설명 글 (OK, NOT FOUND..)

- HTTP 헤더
    - header-field = field-name ":" OWS field-value OWS (OWS : 띄어써도 되고, 안 해도 됨)
    - 예시

        ```java
        Host: www.google.com
        ```

    - 예시2

        ```java
        Content-Type: text/html;charset=UTF-8
        Content-Length: 3423

        //Field Name은 대소문자 구분 없음!
        //field-value는 대소문자 구분 한다!
        ```

    - HTTP 헤더는 HTTP 전송에 필요한 모든 부가 정보가 들어있다!
        - 메시지 바디의 내용, 크기, 압축, 인증, 요청 클라이언트 정보...
        - 필요시, 임의의 헤더 추가 가능 (eg > helloworld: hihi! )

- HTTP 메시지 바디
    - 실제 전송할 데이터가 들어가 있다.
    - HTML 문서, 이미지, 영상, JSON 등.. 모든 byte로 표현할 수 있는 데이터 전송 가능하다.