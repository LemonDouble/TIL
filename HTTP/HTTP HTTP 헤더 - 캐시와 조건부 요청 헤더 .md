# HTTP : HTTP 헤더 - 캐시와 조건부 요청 헤더

분류: HTTP
작성일시: 2021년 7월 21일 오후 3:10

## 1. 캐시의 기본 동작

- 캐시가 없는 경우
    1. Client Request

    ```java
    GET /star.jpg
    ```

    1. Server Respond

        (이 때, HTTP Header : 0.1 Mb, HTTP Body(Star.jpg)는 1Mb 가정)

    ```java
    HTTP/1.1 200 OK
    Content-Type: image/jpeg
    Content-Length: 34012

    .......
    ```

    1. Client Request(중복)

        (똑같은 star.jpg)

    ```java
    GET /star.jpg
    ```

    1. Server Respond(중복)

        똑같은 파일을 재전송하고, 1Mb가 낭비된다

    ```java
    HTTP/1.1 200 OK
    Content-Type: image/jpeg
    Content-Length: 34012

    .......
    ```

- 캐시가 없을 때의 문제점
    - 데이터가 변경되지 않아도, 계속 네트워크를 통해 데이터를 다운받아야 한다.
    - 인터넷 네트워크는 (상대적으로) 매우 느리고, 매우 비싸다!
    - 브라우저 로딩 속도가 느리다
    - 느린 사용자 경험

---

- 캐시를 적용한 경우 (Cache 유효시간 내)
    1. Clinet Request

    ```java
    GET /star.jpg
    ```

    1. Server Respond

        (Cache 헤더 정보 같이 송신, 이 경우 60초 유지)

    ```java
    HTTP/1.1 200 OK
    Content-Type: image/jpeg
    Cache-control: max-age=60
    Content-Length: 34012

    .......
    ```

    1. Client는 응답 결과를 캐시에 저장
    2. Client는 요청 전 캐시를 먼저 확인, Cache에 Star.jpg 있으므로 Cache에 있는 Data 사용
    3. 네트워크 요청 없이도 사용 가능!

- 캐시 적용의 장점
    - 캐시 가능 기간동안 네트워크를 사용하지 않아도 된다.
    - 비싼 네트워크 사용량을 줄일 수 있다
    - 브라우저 로딩 속도가 매우 빠르다
    - 빠른 사용자 경험

- 캐시를 적용한 경우 (Cache 유효시간 초과 - 기본)
    1. Clinet가 Cache 확인했는데, 유효 시간(60초) 초과
    2. Clinet Request
    3. Server Respond (Star.jpg 전송)
    - 만약, Star.jpg가 바뀌지 않았더라도 네트워크 사용해야 한다!
    - 다음의 검증 헤더와 조건부 요청을 통해 문제 해결이 가능하다.

## 2. 검증 헤더와 조건부 요청 - Last-Modified

- 캐시 유효시간 초과시의 상황
    - 서버에서 기존 데이터를 변경 (캐시 =/= 서버 데이터)
    - 서버에서 기존 데이터를 변경하지 않음( 캐시 == 서버 데이터)

- 검증 헤더
    - 캐시 만료 후에도 서버에서 데이터가 변경되지 않은 경우
    - 로컬 캐시를 재사용 할 수 있다!
    - 단, 클라이언트의 데이터와 서버의 데이터가 같다는 사실을 확인할 방법이 필요하다

    - Last-Modified 헤더: 데이터가 마지막으로 수정된 시간

    1. Client Request

        ```java
        GET /star.jpg
        ```

    2. Server Responde 

        (Last-Modified 헤더 추가해서 보내준다)

        ```java
        HTTP/1.1 200 OK
        Content-Type: image/jpeg
        Cache-control: max-age=60
        Last-Modified: Sat, 26-Dec-2020 00:00:00 GMT
        Content-Length: 34012

        .......
        ```

    3. Client는 Last-Modified와 함께 Cache 저장한다!
    4. 만약 Cache-control에 따라 60초 안에 다시 Request 보낸다면 Local Cache 사용한다.
    5. 만약 60초가 초과했다면, Conditional Get 보낸다.

    ```java
    GET /star.jpg
    if-modified-since: Sat, 26-Dec-2020 00:00:00 GMT
    ```

    1. 서버는 Modified 헤더를 통해 판단을 할 수 있다.

        6-1. 만약 수정되지 않았다면 304 Respond 보낸다.

        이 때, HTTP Body는 없다! 헤더만 전송하므로, 네트워크 부담이 크게 줄어든다.

        ```java
        HTTP/1.1 304 Not Modified
        Content-Type: image/jpeg
        Cache-control: max-age=60
        Last-Modified: Sat, 26-Dec-2020 00:00:00 GMT
        Content-Length: 34012

        //HTTP BODY는 없음! 
        ```

        6-2. 만약 수정되었다면, 평소대로 데이터 보낸다.

    2. Client는 만약 304 받았다면 응답 결과(Local Cache Header를 갱신)를 재활용한다.

## 3. 검증 헤더와 조건부 요청 - ETag

- 검증 헤더
    - 캐시 데이터와 서버 데이터가 같은지 검증하는 데이터
    - Last-Modified, ETag
- 조건부 요청 헤더
    - 검증 헤더로 조건에 따른 분기
    - If-Modified-Since : Last-Modified와 함께 사용
    - If-None-Match : ETag 사용
    - 조건이 만족하면 200 OK
    - 조건이 만족하지 않으면 304 Not Modified

- If-Modified-Since + Last-Modified
    - 데이터 미변경 예시
        - 캐시 : 2020/11/10 10:00:00
        - 서버 : 2020/11/10 10:00:00
        - 304 Not Modified, 헤더 데이터만 전송 (BODY 미포함)
        - 전송 용량 0.1Mb (헤더 0.1Mb, 바디 1.0Mb)
    - 데이터 변경 예시
        - 캐시 : 2020/11/10 10:00:00
        - 서버 : 2020/11/10 11:00:00
        - 200 OK, 모든 데이터 전송(BODY 포함)
        - 전송 용량 1.1Mb, (헤더 0.1Mb, 바디 1.0Mb)

- If-Modified-Since, Last-Modified 방법의 단점
    - 1초 미만(0.x초) 단위로 캐시 조정이 불가능
    - 날짜 기반의 로직 사용
    - 만약 데이터를 수정해서, 날짜가 다르지만 같은 데이터를 수정해 데이터 결과가 똑같은 경우 (B → A, A → B)
    - 서버에서 별도의 캐시 로직을 관리하고 싶은 경우
        - 예시로, 만약 Space, 주석처럼 크게 영향이 없는 변경이 생긴 경우, 캐시를 사용하고 싶은 경우

- ETag( Entity Tag)
    - 캐시용 데이터에 임의의 고유한 버전 이름을 달아둠
        - 예) Etag:"v1.0", Etag: "a2jiodwjekjl3" (Hash)
    - 데이터가 변경되면 이 이름을 바꾸어서 변경함 (Hash를 재생성)
        - 예) Etag: "aaaaa" → Etag: "bbbbb"
    - 단순하게 Etag만 보내서 같으면 유지, 다르면 다시 받기!

- Etag + If-None-Math 사용 예시
    1. Client Request

    ```java
    GET /star.jpg
    ```

    1. Server Respond

    ```java
    HTTP/1.1 200 OK
    Content-Type: image/jpeg
    Cache-control: max-age=60
    ETag: "aaaaa"
    Content-Length: 34012

    ...
    ```

    1. Client는 Etag값을 Cache에 저장
    2. Client는 Cache-control 시간 지난 경우, 다음과 같이 Requet 보냄

    ```java
    GET /star.jpg
    If-None-Match: "aaaaa"
    ```

    1. 만약 Etag가 Match되면 304 Not modified

        ```java
        HTTP/1.1 200 OK
        Content-Type: image/jpeg
        Cache-control: max-age=60
        ETag: "aaaaa"
        Content-Length: 34012

        //BODY는 보내지 않는다!
        ```

    2. 만약 Etag가 Match되지 않았다면 200 OK

- Etag의 특징
    - 캐시 제어 로직을 서버에서 완전히 관리
    - 클라이언트는 단순히 Etag를 서버에 제공 (클라이언트는 캐시 메커니즘을 모름)
    - 예시)
        - 서버는 배타 오픈 기간인 3일 동안 파일이 변경되어도 ETag를 동일하게 유지
        - 어플리케이션 배포 주기에 맞춰 ETag 모두 갱신

## 4. 캐시와 조건부 요청 헤더

- 캐시 제어 헤더
    - Cache-Control: 캐시 제어
        - Cache-Control: max-age
            - 캐시 유효 기간, 초 단위
        - Cache-Cotnrol: no-cache
            - 데이터는 캐시해도 되지만, 항상 원(origin) 서버에 검증하고 사용
        - Cache-Control: no-store
            - 데이터에 민감한 정보가 있으므로 저장하면 안 됨
            - 메모리에서만 사용하고 최대한 빨리 삭제

    - Pragma: 캐시 제어(하위 호환)
        - Pragma: no-cache
        - HTTP 1.0 하위 호환
    - Expires: 캐시 유효 기간(하위 호환)
        - expires: Mon, 01 Jan 1990 00:00:00 GMT
        - 만료일을 정확한 날짜로 지정
        - HTTP 1.0부터 사용
        - 지금은 더 유연한 Cache-Control: max-age 권장
        - Cache-Control: max-age와 함께 사용하면 expires는 무시

## 5. 프록시 캐시

- 미국에 있는 원 서버(origin server) 를 한국에서 접속시 500ms (0.5초) 걸린다 가정
- 프록시 캐시 도입 (한국 어딘가에 CDN 비슷한걸 만들어 둠)
    - Client → Proxy Cache Server : 100 ms
    - Proxy Cache Server → Origin Server : 400ms

- Clinet에 있는 Cache : Private Cache
- 프록시 서버에 있는 Cache : Public Cache

- Cache-Control : 캐시 지시어(directives)
    - Cache-Control: public
        - 해당 캐시는 Public 서버에 저장되어도 됨
    - Cache-Control: private (default)
        - 응답이 해당 사용자만을 위한 것, private 캐시에 저장해야 함
    - Cache-Control: s-maxage
        - 프록시 캐시에만 적용되는 max-age
    - Age: 60 (HTTP 헤더)
        - Origin 서버에서 응답 후, 프록시 서버 내부에서 머문 시간 (초)

## 6. 캐시 무효화

- 확실한 캐시 무효화 응답
    - Cache-Control: no-cache, no-store, must-revalidate
    - Pragma: no-cache
        - HTTP 1.0 하위호환

- 위 Header를 전부 넣어줘야 함!
    - 통장 잔고 등, 언제나 갱신될 수 있고 절대 캐시되어선 안 되는 데이터 등에 사용

- Cache-Control : no-cache
    - 데이터는 캐시해도 되지만, 항상 원 서버에 검증하고 사용!
- Cache-Control: no-store
    - 데이터에 민감한 정보가 있으므로 저장하면 안 됨 (메모리에서 사용하고, 가능한 빨리 삭제)
- Cache-Control: must-revalidate
    - 캐시 만료 후 최초 조회시, 원 서버에 검증해야 함
    - 원 서버 접근 실패시, 반드시 오류가 발생해야 함 -504(Gateway Timeout)
    - must-revalidate는 캐시 유효 시간 내라면 캐시를 사용함
- Pragma: no-cache
    - HTTP 1.0 하위호환

- No-Cache를 사용하면 원 서버에 검증하는데, 왜 must-revalidate 헤더도 사용하는가?
    - 만약 중간 프록시 서버가 순간적으로 원 서버에 접근할 수 없을 경우, Old Cache 보내주는 경우가 있다
    - must-revalidate 사용하면, 원 서버에 접근할 수 없을 때 504 Error가 보증된다