# HTTP : HTTP 메서드 - 이론

분류: HTTP
작성일시: 2021년 7월 20일 오후 2:05

## 1. HTTP API 설계

- 일단 무작정 설계해보자!
    - 회원 목록 조회 : /read-member-list
    - 회원 조회 : /read-member-by-id
    - 회원 등록 : /create-member
    - 회원 수정 : /update-member
    - 회원 삭제 : /delete-member

- 하지만 위와 같은 URI는 좋은 설계가 아니다.
    - URI에서 가장 중요한 것은 리소스 식별

- 리소스란?
    - 회원 등록/수정/조회는 리소스가 아니다!
    - "회원" 자체가 하나의 리소스!

- 리소스를 어떻게 식별하는게 좋을까?
    - 회원을 등록, 수정, 조회하는 것을 모두 배제
    - 회원이라는 리소스 자체만 식별하면 된다.
    - 즉, 회원 리소스를 URI에 매핑!

---

- URL 계층 구조 활용!
    - 회원 목록 조회 :  /members
    - 회원 조회 : /members/{id}
    - 회원 등록 : /members/{id}
    - 회원 수정 : /members/{id}
    - 회원 삭제 : /members{id}

- 조회,등록,수정,삭제는 URI가 같지만, HTTP Method를 통해 구분!

---

- HTTP 주요 메서드
    - GET : 리소스 조회
    - POST : 요청 데이터 처리, 주로 등록에 사용
    - PUT : 리소스를 대체, 해당 리소스가 없으면 생성 (덮어쓰기)
    - PATCH : 리소스 부분 변경 ( 이름을 바꾸거나.. )
    - DELETE : 리소스 삭제

- HTTP 기타 메소드
    - HEAD : GET과 동일하지만 BODY 부분 제외하고, 상태 줄과 헤더만 반환
    - OPTIONS : 대상 리소스에 대한 통신 가능 옵션 (메서드) 를 설명, (주로 CORS에서 사용)
    - CONNECT : 대상 자원으로 식별되는 서버에 대한 터널을 설정 (사용 적음)
    - TRACE : 대상 리소스에 대한 경로를 따라 메세지 루프백 테스트 수행  (사용 적음)

## 2. HTTP Method - Get, Post

```java
GET /search?q=hello&hl=ko HTTP/1.1
Host: www.google.com
```

- GET : 리소스 조회
    - 서버에 전달하고 싶은 데이터는 query(쿼리 파라미터, 쿼리 스트링) 을 통해 전달
    - 메세지 BODY 통해 데이터 전달이 가능은 하지만, 지원하지 않는 곳이 많아 권장하지 않음

    - 예시
        1. Request (100번째 member의 리소스를 조회하겠다)

        ```java
        GET /members/100 HTTP/1.1
        Host: www.aaa.com
        ```

        1. Response

        ```java
        HTTP/1.1 200 OK
        Content-Type: application/json
        Content-Length: 34

        {
        	"username": "young",
        	"age": 20
        }
        ```

---

```java
POST /members HTTP/1.1
Content-Type: application/json

{
	"username": "hello",
	"age":20
}
```

- POST : 요청 데이터 처리
    - 메세지 BODY를 통해 서버로 요청 데이터 전달
    - 서버는 요청 DATA를 처리
    - 주로 전달된 데이터로 신규 리소스 등록, 프로세스 처리 등에 사용

    - 예시
        1. Request (member로 아래 데이터를 등록해 달라)

        ```java
        POST /members HTTP/1.1
        Content-Type: application/json

        {
        	"username": "young",
        	"age": 20
        }
        ```

        1. Response (member로 등록한 후, 반환)

        ```java
        HTTP/1.1 201 Created
        Content-Type: application/json
        Content-Length: 34
        Location: /members/100
        //새로 생성된 데이터의 URI를 보내준다.

        {
        	"username": "young",
        	"age": 20
        }
        ```

- POST의 여러가지 의미
    - SPEC : 대상 리소스가 리소스의 고유한 의마 체계에 따라 요청에 포함 된 표현을 처리
    - 즉, Resource의 처리 방법은 정해진 내용이 없다!
    - 예시
        - HTML 양식에 입력된 데이터 필드를 처리 프로세스에 제공
            - Form에 작성된 정보로 회원 가입, 주문 등에 사용
        - 게시판, 뉴스 그룹, 메일링 리스트 등에 메세지 게시
            - 게시판 글쓰기, 댓글달기
        - 서버가 아직 식별하지 않은 새 리로스 생성
            - 신규 주문 생성
        - 기존 자원에 데이터 추가
            - 한 문서 끝에 데이터 생성

- POST 정리
    1. 새 리소스 생성(등록)  
        - 서버가 아직 식별하지 않은 새 리소스 생성
    2. 요청 데이터 처리
        - 단순히 데이터 생성, 변경을 넘어 프로세스를 처리하는 경우
        - 예 ) 주문 → 결제완료 → 배달시작 → 배달완료 처럼 값 변경을 넘어 프로세스의 상태가 변경되는 경우도 POST로 처리
        - POST의 결과로 새로운 리소스가 생성되지 않을 수도 있음
        - 예) POST /orders/{orderId}/start-delivery (Control URL)
    3. 다른 메서드로 처리하기 애매한 경우
        - 예 ) JSON으로 조회 데이터 넘겨야 하는데, GET 사용하기 어려운 경우
        - (GET Method는 Body에 값을 넣는걸 지원하지 않는 서버가 많으므로)
        - 가능한 GET을 쓰는게 좋지만, 정말 어쩔 수 없는 경우 POST로 처리 가능하다.

## 3. HTTP Method - PUT, PATCH, DELETE

```java
PUT /members/100 HTTP/1.1
//Client가 Resource의 위치를 정확히 식별하고 있다
Content-Type: application/json

{
	"username": "hello",
	"age": 20
}
```

- PUT
    - 리소스를 대체 (파일 덮어쓰기 생각하면 된다!)
        - 리소스가 있으면 대체
        - 리소스가 없으면 생성
    - 중요! 클라이언트가 리소스를 식별
        - 클라이언트가 리소스 위치를 알고 URI를 지정함
        - POST와의 큰 차이점! (POST는 /members URI 사용했다)

- PUT은 리소스를 완전히 대체한다!
    - 만약 원래 서버에 있던 정보가 아래와 같을 때

    ```java
    {
    	"username": "hello",
    	"age": 10
    }
    ```

    - 다음과 같이 PUT 요청 보냈을 경우

    ```java
    PUT /members/100 HTTP/1.1
    //Client가 Resource의 위치를 정확히 식별하고 있다
    Content-Type: application/json

    {
    	"age": 20
    }
    ```

    - 서버에 있는 Data의 username 필드가 사라진다! (완전히 대체)

    ```java
    {
    	"age": 20
    }
    ```

---

- PATCH : 리소스 부분 변경
    - PUT과 같지만, 리소스를 부분 변경한다!

- 가끔.. PATCH 안 되는 경4. 우 POST 사용해도 된다

---

```java
DELETE /members/100 HTTP/1.1
Host: www.google.com
```

- DELETE : 데이터 제거시 사용한다

## 4. HTTP Method의 속성

- 안전 (Safe Methods)
    - 호출해도 리소스가 변경되지 않는다.

    - Safe : GET
    - UnSafe : POST, PUT, DELETE, PATCH

---

- 멱등 (Idempotent Methods)
    - f(f(x)) = f(x)
    - 한번 호출하든, 두번 호출하든, 100000000 번 호출하든 결과가 똑같다.

    - 멱등 메서드
        - GET : 한번 조회하듯, 두번 조회하든 같은 결과가 조회된다.
        - PUT : 결과를 대체한다. 같은 데이터로 여러번 대체해도 최종 결과는 같다.
        - DELETE : 결과를 삭제한다. 같은 요청을 여러번 해도 삭제된 결과는 똑같다.
    - 비 멱등 메서드
        - POST : 만약 두번 호출하면, 같은 결제가 중복해서 발생할 수 있다!

- 멱등을 왜 사용하는가?
    - 자동 복구 매커니즘
    - 서버가 TIMEOUT 등으로 정상 응답이 돌아오지 않았을 때, 클라이언트가 같은 요청을 다시 해도 되는가?

- 재요청 도중 다른 곳에서 리소스가 바뀐다면?
    - USER 1 : GET → username: A, age : 20
    - USER 2 : PUT → username: B, age : 30
    - USER1 : GET → username: B, age : 30
    - 멱등은 외부 요인으로 중간에 리소스가 변경되는 것 까지는 고려하진 않는다!
    - 

---

- 캐시 가능 (Cacheable Methods) (중요!)
    - 응답 결과 리소스를 Cache해서 사용해도 되는가?

    - Cache 가능
        - 이론상 : GET, HEAD, POST, PATCH는 CACHE 가능
        - 실제 : GET, HEAD 정도로만 캐시로 사용
            - GET, HEAD는 URL을 Key로 사용하면 되므로 Cache하기 쉽다.
            - POST, PATCH는 본문 내용까지 Cache Key로 고려해야 하는데, 구현하기 힘들다.