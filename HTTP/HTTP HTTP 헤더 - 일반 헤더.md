# HTTP : HTTP 헤더 - 일반 헤더

분류: HTTP
작성일시: 2021년 7월 20일 오후 8:07

## 1. HTTP 헤더 개요

- Header-fied = field-name ":" OWS field-value OWS (OWS : 띄어쓰기 허용)
- field-name은 대소문자 구분 없음

```java
GET /search?q=hello&hl=ko HTTP/1.1
Host: www.google.com  // < 이 부분!
```

```java
HTTP/1.1 200 OK
Content-Type:text/html;charset=UTF-8 // < 이 부분!
Content-Length: 3423                 // < 이 부분!

...

```

- 용도
    - HTTP 전송에 필요한 모든 부가정보
        - 예) 메시지 바디의 내용, 크기, 압축, 인증, 요청 클라이언트, 서버 정보, 캐시 관리 정보 등..
    - 표준 헤더가 엄청 많다!
    - 필요시 임의의 헤더 추가가 가능하다
        - helloworld: hihi

- 헤더 분류(과거 : RFC2616, 폐기됨)
    - General 헤더 : 메시지 전체에 적용되는 정보
        - 예) Connection: close
    - Request 헤더 : 요청 정보
        - 예) User-Agent: Mozila/5.0 (Machintosh; ...)
    - Response 헤더 : 응답 정보
        - 예) Server: Apache
    - Entity 헤더 : 엔티티 바디 정보
        - 예) Content-Type: text/html, Content-Length: 3423
    - 메세지 본문 (Message Body)는 엔티티 본문 (Entity Body) 를 전달하는데 사용
    - 엔티티 본문(Entity Body) 는 요청/응답에서 전달할 실제 데이터
    - 엔티티 헤더는 엔티티 본문의 데이터를 해석할 수 있는 정보 제공
        - 데이터 유형(HTML, JSON), 데이터 길이, 압축 정보 등등

- 현재의 헤더 분류 ( RFC 7230~7235, 2014년)
    - 엔티티(Entity) → 표현 (Representation)
    - Representation = Representation 메타 데이터 + Representation 데이터

    - 메세지 본문(Message Body) 를 통해 표현 데이터 전달
    - Message Body = Payload
    - 표현은 요청/응답에서 전달할 실제 데이터
    - 표현 헤더는 표현 데이터를 해석할 수 있는 정보 제공
        - 데이터 유형(HTML, JSON..) , 데이터 길이, 압축 정보 등

## 2. 표현 헤더

- 표현 헤더는 Request, Response 둘 다 사용

- Content-Type: 표현 데이터의 형식 설명
    - 미디어 타입, 문자 인코딩 정보
    - 예시)
        - text/html; charset=utf-8
        - application/json (기본:utf-8)
        - image/png
- Content-Encoding: 표현 데이터의 압축 방식, 인코딩 방식
    - 표현 데이터를 압축하기 위해 사용
    - 데이터를 전달하는 곳에서 압축 후 인코딩 헤더 추가
    - 데이터를 읽는 쪽에서 인코딩 헤더의 정보로 압축 해제
    - 예시)
        - gzip
        - deflate
        - identiiy

- Content-Language: 표현 데이터의 자연 언어
    - 예시)
        - ko
        - en
        - en-US

- Content-Length: 표현 데이터의 길이
    - Byte 단위
    - Transfer-Encoding(전송 코딩) 을 사용하면 Content-Length를 사용하면 안 됨

## 3. 협상 헤더 (Content-Negotiation)

- 클라이언트가 선호하는 표현 요청

- 다음과 같은 상황의 경우..
    - Clinet : 한글 브라우저를 사용
    - Server : Multi-Language 지원 서버 ( en, ko)

    - Negotiation Header 적용 전
        - Server는 기본 Language인 English 반환

    - Negotiation Header 적용 후
        - Clinet는 Request 할 때 Accept-Language: ko 헤더를 담아 요청한다.
        - Server는 Accept-Language 헤더를 확인하고, ko 데이터를 넣어 반환한다.

- 협상 헤더의 종류
    - Accept: 클라이언트가 선호하는 미디어 타입 전달
    - Accept-Charset: 클라이언트가 선호하는 문자 인코딩
    - Accept-Encoding: 클라이언트가 선호하는 압축 인코딩
    - Accept-Language: 클라이언트가 선호하는 자연 언어

    - 협상 헤더는 Request시에만 사용

- 협상 헤더와 우선순위
    1. Quality Values(q)
        - 0~1, 클수록 높은 우선순위
        - 생략하면 1
        - 예시)
            - Accept-Language: ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7
                1. ko-KR;q=1(q 생략)
                2. ko;q=0.9
                3. en-US;q=0.8
                4. en;q=0.7
            - Server는 위와 같은 헤더 해석하여, ko-KR → ko → en-US → en 순으로 보내주게 된다!
    2. 구체적인 타입이 우선한다!
        - 예시) Accept: text/*, text/plain, text/pain;format=flowed, * / *
            - 이런 경우, 구체적인 Type이 우선순위가 높다!
                1. text/plain; format=flowed
                2. text/plain
                3. text/*
                4. *  / *

    3.  구체적인 것을 기준으로 미디어 타입을 맞춘다

    - 예시) Accept : text/*;q=0.3, text/html;q=0.7, text/html;level=1, text/html;level=2;q=0.4,* / *; q=0.5
        1. text/html;level=1 → 1.0
        2. text/html → 0.7
        3. text/plain → 0.3 (text/* 에 Match)
        4. image/jpeg → 0.5
        5. text/html;level=2 → 0.4
        6. text/html;level=3 → 0.7(text/html 에 Match)

## 4. 전송 방식

- 전송 방식의 종류
    - 단순 전송
        - Content-Length

        ```java
        HTTP/1.1 200 OK
        Content-Type: text/html;charset=UTF-8
        Content-Length: 3423

        ....
        ```

    - 압축 전송
        - Content-Encoding
        - Content-Length

        ```java
        HTTP/1.1 200 OK
        Content-Type: text/html;charset=UTF-8
        Content-Encoding: gzip
        Content-Length: 521

        ....
        ```

    - 분할 전송 (Data가 큰 경우!)
        - Transfer-Encoding: chunked
        - Content-Length 사용하면 안 된다! (전체 크기 예측불가하고, 각 Chunk마다 크기 있으므로)

        ```java
        HTTP/1.1 200 OK
        Content-Type: text/plain
        Transfer-Encoding: chunked

        5
        Hello
        5
        World
        0
        \r\n
        ```

    - 범위 전송 (일부 range만 지정)
        - 만약 다운받다가 중간에 끊겼다면?

        ```java
        //Request
        GET /event
        Range: bytes=1001-2000

        //Response
        HTTP/1.1 200 OK
        Content-Type: text/plain
        Content-Range: bytes 1001-2000/2000

        ....
        ```

## 5. 일반 정보

- 일반 정보 헤더의 종류
    - From : 유저 에이전트의 이메일 정보
        - 일반적으로 잘 사용되진 않음
        - 검색엔진 같은 곳에서 주로 사용
        - Request에서 사용
    - Refer : 이전 웹 페이지 주소
        - 현재 요청된 페이지의 이전 웹 페이지 주소
        - A→ B로 이동하는 경우, B를 요청할 때 Referer: A를 포함해서 요청
        - Referer을 사용해 유입 경로 분석이 가능
        - Request에서 사용
        - 참고) referer는 단어 referrer의 오타 : 이미 많이 사용해서 바꾸지 못한다..
    - User-Agent: 유저 에이전트 애플리케이션 정보
        - user-Agent: Mozila/5.0 (Machintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.183 Safari/537.36
        - 클라이언트의 어플리케이션 정보 (웹 브라우저 정보, 등등)
        - 통계 정보
        - 어떤 종류의 브라우저에서 장애가 발생하는지 파악 가능
        - Request에서 사용
    - Server: 요청을 처리하는 오리진 서버의 소프트웨어 정보
        - 오리진 서버 : 내 Request에 대해 Representation을 만들어주는 진짜 서버 (Cache 서버 등등 제외하고..)
        - Server: Apache/2.2.22 (Debian)
        - Server: nginx
        - Response에서 사용
    - Date: 메시지가 생성된 날짜와 시간
        - Date: Tue, 15 Nov 1994 08:12:31 GMT
        - Response에서 사용

## 6. 특별한 정보

- 특별한 정보 헤더의 종류
    - Host: 요청한 호스트 정보(도메인)
        - 필수
        - 하나의 서버가 여러 도메인을 처리해야 할 때
        - 하나의 IP 주소에 여러 도메인이 적용되어 있을 때
        - Host 필드를 통해 어떤 도메인에서 왔는지 알 수 있음!
        - Request에서 사용
    - Location: 페이지 리다이렉션
        - 브라우저는 3xx Response 결과에 Location 결과가 있으면, 해당 위치로 자동 이동
        - 201 (Created)에서도 사용 → 요청에 의해 생성된 URI
        - Response에서 사용
    - Allow: 허용 가능한 HTTP 메서드
        - 405 (Method Not Allow) 에서 응답에 포함해야 함
        - Allow: GET, HEAD, PUT ( GET, HEAD, PUT만 서버에서 지원합니다!)
        - Response에서 사용
    - Retry-After: 유저 에이전트가 다음 요청을 하기까지 기다려야 하는 시간
        - 503 (Service Unavailable) : 서비스가 언제까지 불능인지 알려줄 수 있음
        - Retry-After: Fri, 31 Dec 1999 23:59:59 GMT (날짜 표기)
        - Retry-After: 120 (초단위 표기)

## 7. 인증 헤더

- 인증 헤더의 종류
    - Authorization: 클라이언트 인증 정보를 서버에 전달
        - Authorization: Basic xxxxxxxxxxx......
        - 인증 방법마다 다르다
    - WWW-Authenticate: 리소스 접근시 필요한 인증 방법 정의
        - 401 Unauthorized 응답과 함꼐 사용
        - WWW-Authenticate: Newauth realm="apps, type=1, title="Login to \"apps\""

            , Basic realm="simple"

## 8. 쿠키

- HTTP Stateless
    - HTTP는 Stateless 프로토콜!
    - Client와 Server가 요청과 응답을 주고 받으면 연결이 끊어진다. (Persistent connection 있지만, 기본 Concpet는)
    - 클라이언트가 다시 요청해도, 서버가 이전 요청을 기억하지 못한다.
    - 클라이언트와 서버는 서로 상태를 유지하지 않는다.
    - 해결하기 위해 쿠키가 필요!

- 쿠키
    - 예) set-cookie: sessionId=abcde1234; expires=Sat, 26-Dec-2020 00:00:00 GMT; path=/; domain=.google.com; Secure
    - 주 사용처
        - 사용자의 로그인 세션 관리
        - 광고 정보 트래킹
    - 쿠키 정보는 항상 서버에 전송됨
        - 네트워크 트래픽 추가 유발
        - 따라서, 최소한의 정보만 사용(세션id, 인증 토큰)
        - 서버에 전송하지 않고, 웹 브라우저 내부에만 데이터를 저장하고 싶으면 웹 스토리지 (localStorage, sessionStorage) 참고
    - 주의!
        - 보안에 민감한 데이터는 저장하면 안 됨(주민번호, 신용카드 번호 등)

- 쿠키-LifeCycle
    - Set-Cookie: expires=Sat, 26-Dec-2020 00:00:00
        - 만료일이 되면 자동으로 쿠키 삭제
    - Set-Cookie: max-age=3600 (3600초)
        - 0이나 음수를 지정하면 쿠키 삭제
    - 세션 쿠키 : 만료 날짜를 생략하면 브라우저 종료시까지만 유지
    - 영속 쿠키 : 만료 날짜를 입력하면 해당 날짜까지 유리

- 쿠키- Domain
    - 예시) Domain=example.org
    - 명시: 명시한 문서 기준 도메인 + 서브 도메인 포함
        - domain=example.org를 지정해서 쿠키 생성
            - example.org는 물론이고
            - dev.example.org도 쿠키 접근
    - 생략: 현재 문서 기준 도메인만 적용
        - example.org에서 쿠키를 생성하고, domain 지정을 생략
            - example.org에서만 쿠키 접근
            - dev.example.org는 쿠키 미접근

- 쿠키-Path
    - 예) Path=/home
    - 이 경로를 포함한 하위 경로 페이지만 쿠키 접근
    - 일반적으로 path=/ 루트로 지정
    - 예시)
        - path=/home 지정
        - /home (O)
        - /home/level1 (O)
        - /home/level1/level2 (O)
        - /hello (X)

- 쿠키-보안
    - Secure
        - 쿠키는 http, https를 구분하지 않고 전송
        - Secure를 적용시 https인 경우만 전송
    - HttpOnly
        - XSS 공격 방지 (공격하려는 사이트에 Script를 심는 기법)
        - Javascript에서 접근 불가(document.cookie)
        - HTTP 전송에만 사용
    - SameSite (최신 기능,
        - XSRF(CSRF) 공격 방지
        - 요청 도메인과 쿠키에 설정된 도메인이 같은 경우에만 쿠키 전송

- 쿠키 헤더의 종류
    - Set-Cookie: 서버에서 클라이언트로 쿠키 전달(응답)
    - Cookie: 클라이언트가 서버에서 받은 쿠키를 저장하고, HTTP 요청시 서버로 전달