# HTTP : HTTP 상태 코드

분류: HTTP
작성일시: 2021년 7월 20일 오후 5:25

## 1. HTTP 상태 코드 소개

- 상태 코드
    - 클라이언트가 보낸 요청의 처리 상태를 응답해서 알려주는 기능

    - 1xx (Informational) : 요청이 수신되어 처리중 (거의 사용되지 않음)
    - 2xx (Successful) : 요청 정상 처리
    - 3xx (Redirection) : 요청이 완료되려면 추가 행동이 필요
    - 4xx (Client Error) : 클라이언트 오류, 잘못된 문법등으로 서버가 요청을 수행할 수 없음.
    - 5xx (Server Error) : 서버 오류, 서버가 정상 처리하지 못함

- 만약 모르는 상태 코드가 나타나면?
    - Status Code는 추가될 수도 있음
    - 이런 경우, Clinet는 상위 상태 코드로 해석해 처리
    - 이런 경우, 미래에 새로운 상태 코드가 추가되어도 클라이언트가 변경되지 않아도 됨

        (물론 패치하면 더 좋다)

    - 예시)
        - 299 ??? → 2xx (Successful) : 잘 모르겠지만 몬가 잘 됐나보다..
        - 451 ??? → 4xx (Client Error) : 잘은 모르지만 내가 잘못했구나..
        - 큰 범위로 치환하여 해결한다!

- 모든 Status Code를 다 사용하는게 좋나요?
    - 사용하는 Code를 개발 전 범위를 정해놓고, 스펙에 맞게 사용하는 것이 좋다.
    - 모든 Code를 다 사용하는 경우, 오히려 혼란스러울 수 있다.

## 2. 2xx - Success

- 요청 성공!

- 200 : OK (요청 성공)
    - 예시 : GET 조회한 경우, 결과 DATA 돌려주며 200 OK
- 201 : Created (요청 성공해서 새로운 리소스가 생성됨)
    - 예시 : POST로 생성 요청해서, 결과( URI ) 돌려주며 201 Created
- 202 : Accepted (요청이 접수되었지만, 처리가 완료되지 않았음)
    - Batch 처리 같은 곳에서 사용
    - 예시 : 요청 접수 후, 1시간 뒤에 Batch Process가 요청을 처리하는 경우
- 204 : No Content (서버가 요청을 성공적으로 수행했지만, 응답 Payload 본문에 보낼 데이터가 없음)
    - 예시) 웹 문서 편집기에서 Save 버튼
    - Save 버튼의 결과로 아무 내용이 없어도 된다. (save가 됐는지만 확인하면 된다!)
    - Save 버튼을 눌러도 같은 화면을 유지해야 한다.
    - 결과 내용이 없어도, 204 메세지만으로 성공을 인식할 수 있다.

## 3. 3xx - Redirection

- 요청을 완료하기 위해 유저 에이전트의 추가 조치가 필요하다!

- Redirection : 웹 브라우저가 3xx 응답의 결과에 Location 헤더가 있으면, Location 위치로 자동 이동한다.
    - 예시)
        1. Client Request

        ```java
        GET /event HTTP/1.1
        Host: www.aaa.com
        ```

        1. Server Respond ( 페이지가 이동했다! 새 페이지 주소는 new-event이다 )

        ```java
        HTTP/1.1 303 Moved Permantly
        Location: /new-event
        ```

        1. Client Request (자동으로 새로운 page로 Redirect)

        ```java
        GET /new-event HTTP/1.1
        Host: www.aaa.com
        ```

        1. Server Respond...

- Redirection의 종류
    - 영구 리다이렉션 : 특정 리소스의 URI가 영구적으로 이동
        - 예시) /members → /users
        - 예시) /event → /new-event
    - 일시 리다이렉션 : 일시적인 변경
        - 주문 완료 후 주문 내역 화면으로 이동
        - PRG: Post/Redirection/Get
    - 특수 리다이렉션
        - 결과 대신 캐시를 사용
        - Client → Server : 캐시 있는데 이거 사용해도 됨?
        - Server → Clinet : ㅇㅇ 써도됨

- 영구 리다이렉션
    - 원래의 URL을 사용 X, 검색 엔진 등에서도 변경 인지

    - 301 Moved Permanetly
        - 리다이렉트 요청 시 메서드가 GET으로 변하고, 본문이 제거될 수 있음 (may)
        - 항상 제거되진 않고, 브라우저마다 다를 수 있다. (대부분은 GET으로 바뀐다)

        ```java
        //1. Client Request 
        POST /event HTTP/1.1
        Host: www.aaa.com

        name=hello&age=20 // << BODY DATA!

        //2. Server Reply (Redirect)
        HTTP/1.1 301 Moved Permanetly
        Location: /new-event 

        //3. Client Redirect (BODY 정보 사라짐)
        GET /new-event HTTP/1.1
        Host: www.aaa.com

        //4. Clinet Response..
        ```

    - 308 Permanent Redirect
        - 301과 기능은 같음
        - 리다이렉트 요청 시 메서드와 본문 유지

        ```java
        //1. Client Request 
        POST /event HTTP/1.1
        Host: www.aaa.com

        name=hello&age=20 // << BODY DATA!

        //2. Server Reply (Redirect)
        HTTP/1.1 308 Permanent Redirect
        Location: /new-event 

        //3. Client Redirect (POST, BODY DATA도 유지)
        POST /new-event HTTP/1.1
        Host: www.aaa.com

        name=hello&age=20 // << BODY DATA 유지!

        //4. Clinet Response..
        ```

- 일시적 리다이렉션
    - 리소스의 URI가 일시적으로 변경됨
    - 따라서, 검색 엔진 등에서 URL을 변경하면 안 됨.

    - 302 Found
        - 리다이렉트시 요청 메서드가 GET으로 변하고, 본문이 제거될 수 있음(may)
    - 307 Temporary Redirect
        - 302와 기능은 같음.
        - 리다이렉트시 요청 메서드와 본문 유지 (요청 메서드를 절대 변경하면 안 된다!)
    - 303 See Other
        - 302와 기능은 같음
        - 302는 GET으로 변할수도, 안 할수도 있음.
        - 리다이렉트시 요청 메서드가 반드시 GET으로 변경

- 실제로는 302를 많이 사용하긴 한다.. (Framework 등..)
- 자동 리다이렉션 시, GET으로 변해도 되면 302 사용해도 큰 문제는 없다

- PRG: Post/Redirect
    - 일시적인 리다이렉션의 예시
    - POST로 주문 후, 웹 브라우저를 새로고침하면?
    - 새로고침은 다시 요청, 중복 주문이 들어갈 수 있다.

    - PRG 사용 전 예시

        ![HTTP%20HTTP%20%E1%84%89%E1%85%A1%E1%86%BC%E1%84%90%E1%85%A2%20%E1%84%8F%E1%85%A9%E1%84%83%E1%85%B3%204d9de871d14c43b7a3a5baffba69fbf9/Untitled.png](HTTP%20HTTP%20%E1%84%89%E1%85%A1%E1%86%BC%E1%84%90%E1%85%A2%20%E1%84%8F%E1%85%A9%E1%84%83%E1%85%B3%204d9de871d14c43b7a3a5baffba69fbf9/Untitled.png)

        1. POST로 Server에 주문 요청
        2. Server는 DB에 주문 넣고 처리
        3. Server는 200을 Respond
        4. Clinet가 새로고침 ( 렉이 걸리거나 하는 등의 이유로.. ) → Server에 다시 POST로 주문 요청
        5. Server는 DB에 주문 넣고 처리
        6. Server는 200을 Respond

        - 결과: 사용자는 새로고침을 한 번 했을 뿐인데, 주문이 2번 됐다 (중복 주문!)
            - 서버에서 주문번호 등으로 중복을 찾아 Drop할 수 있지만, 결제(돈) 과 관련된 문제는 여러 안전장치를 세팅하는 것이 좋다.
            - Clinet단에서도 안전장치가 있는 것이 좋다.

    - PRG 사용 후
        - POST로 주문 후, 새로 고침으로 인한 중복 주문 방지
        - POST로 주문 후, 주문 결과 화면을 GET 메서드로 리다이렉트
        - 새로고침해도 결과 화면을 GET으로 조회
        - 중복 주문 대신, 결과 화면만 GET으로 다시 요청

        ![HTTP%20HTTP%20%E1%84%89%E1%85%A1%E1%86%BC%E1%84%90%E1%85%A2%20%E1%84%8F%E1%85%A9%E1%84%83%E1%85%B3%204d9de871d14c43b7a3a5baffba69fbf9/Untitled%201.png](HTTP%20HTTP%20%E1%84%89%E1%85%A1%E1%86%BC%E1%84%90%E1%85%A2%20%E1%84%8F%E1%85%A9%E1%84%83%E1%85%B3%204d9de871d14c43b7a3a5baffba69fbf9/Untitled%201.png)

        1. POST로 Server에 주문 요청
        2. Server는 DB에 주문 넣고 처리
        3. Server는 302 Found, Location: /order-result/19 를 Respond
        4. Client는 302를 통해 /order-result/19 를 GET으로 자동 Request
        5. Server는 DB에서 order-result/19 데이터 찾아 Respond
        6. Clinet가 새로고침을 눌러도, 다시 GET /order-result/19 로 이동하므로 중복 주문 X

- 기타 리다이렉션
    - 300 : 안 쓴다
    - 304 : Not Modified
        - 캐시를 목적으로 사용
        - 클라이언트에게 리소스가 수정되지 않았음을 알려준다.
        - 따라서, 클라이언트는 로컬 PC에 저장된 캐시를 재사용한다. (캐시로 Redirect 한다)
        - 304 응답은, 응답에 메세지 바디를 포함하면 안 된다. (Local Cache 사용해야 하므로)
        - Conditional Get, Head 요청시 사용

## 4. 4xx - Clinet Error

- 클라이언트가 잘못함!
- 중요) 클라이언트가 이미 잘못된 요청, 데이터를 보내고 있기 때문에 똑같이 재시도해도 실패함

- Clinet Error
    - 400 Bad Request : 클라이언트가 잘못된 요청을 해 서버가 처리할 수 없음
        - 요청 구문, 메시지 등등이 오류
        - 클라이언트는 요청 내용을 재검토하고 보내야 함.
        - 예시) 요청 파라미터가 잘못되거나, API 스펙이 맞지 않는 경우

    - 401 Unauthorized : 클라이언트가 해당 리소스에 대한 인증이 필요함
        - 인증(Authentication) 되지 않음
        - 401 오류 발생시 응답에 WWW-Authenticate 헤더와 함께 인증 방법을 설명
        - 참고
            - 인증(Authentication) : 본인이 누구인지 확인 (로그인)
            - 인가(Authorization) : 권한부여 (Admin 권한처럼 특정 리소스에 접근할 수 있는 권한, 인증을 한 후에야 인가를 할 수 있음)
            - 401 오류는 인증(Authentication) 과 관련된 문제! (이름이 좀 잘못된 것 같다..)

    - 403 Forbidden : 서버가 요청을 이해했지만, 승인을 거부함
        - 주로 인증 자격 증명은 있지만, 접근 권한이 불충부한 경우
        - 예) Admin 아닌 사용자가 로그인은 했지만, Admin 등급 리소스에 접근하는 경우

    - 404 Not Found : 요청 리소스를 찾을 수 없음
        - 요청 리소스가 서버에 없음
        - 또는 클라이언트가 권한이 부족한 리소스에 접근하는데, 해당 리소스를 숨기고 싶을 때

## 5. 5xx - Server Error

- 서버 문제로 오류 발생함!
- 서버에 문제가 있기 떄문에, 재시도하면 성공할 수도 있음 (서버가 복구되거나 하는 경우)

- Server Error
    - 500 Internal Server Error : 서버 문제로 오류 발생
        - 서버 내부 문제로 오류 발생
        - 분류하기 애매하면 일단 500으로..

    - 503 Service Unavailable
        - 서버가 일시적 과부하, 또는 예정된 작업으로 잠시 요청을 처리할 수 없음
        - Retry-After 헤더 필드로 얼마 뒤에 복구되는지 보낼 수도 있음.

- 500대 Error는 서버가 진짜 문제가 있을 때만 사용하도록 한다!
    - 예시 ) 결제 시스템에서 잔액 없음 → 500 Error 내면 안 된다!
    - 비즈니스 로직 상의 예외 케이스이지, 서버 자체가 문제가 생기는 것은 아니다!

    - DB가 내려가거나, Exception 발생하거나, 정말 문제 생겼을 때만 500 ERROR 발생!