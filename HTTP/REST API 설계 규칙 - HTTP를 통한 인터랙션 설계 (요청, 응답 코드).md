# REST API 설계 규칙 - HTTP를 통한 인터랙션 설계 (요청, 응답 코드)

분류: HTTP
작성일시: 2021년 9월 16일 오후 10:06

### HTTP/1.1

- REST API는 HTTP 1.1의 모든 측면을 수용한다.

- HTTP 1.1 Message의 예시

![Untitled](https://github.com/LemonDouble/TIL/blob/main/HTTP/img/Untitled%204.png)

### 요청 메서드

- Request-Line : HTTP Message의 첫 줄
  - 상호작용하는 메서드를 정의한다.
  - eg> POST / HTTP/1.1
- 이때, 각 Method는 정해진 역할이 있다.

  - GET : 리소스 상태의 표현( 즉, 리소스의 현 상태를 얻을 때) 요청
  - HEAD : 리소스 상태에 대한 메타데이터를 얻을 때
  - PUT : 새로운 리소스를 스토어에 추가, 기존 리소스를 갱신
  - DELETE : 부모에서 리소스를 제거
  - POST : 컬렉션에 새로운 리소스를 만들거나, 컨트롤러를 실행할 때 사용

- 여러 규칙들

  - GET 메소드나 POST 메서드를 사용해서 다른 요청 메서드를 처리해서는 안 된다.

    - REST API 설계를 위해서는 GET이나 POST에 모든 것을 다 때려박으면 안 된다!
    - GraphQL에서는 POST로 모든 걸 처리할 수도 있지만, 지금은 REST API이므로..

  - GET 메소드는 리소스의 상태 표현을 얻는 데 사용해야 한다.

    - GET 요청 메세지는 바디 없이 헤더로만 구성된다.
    - 클라이언트에서 GET 요청을 반복해도 문제가 없어야 한다!
    - 캐시는, 리소스를 제공하는 원래 서버와 통신하지 않고도 캐시된 내용을 제공할 수 있어야 한다!

  - 응답 헤더를 가져 올 때는 반드시 HEAD Method를 사용해야 한다.

    - HEAD Method는 GET Method와 동일한 응답을 주지만, Body가 없다.
    - 클라이언트는 HEAD 메서드를 사용해 리소스 존재 여부를 확인하거나, 메타데이터만 읽을 수 있다.

  - PUT 메소드는 리소스를 삽입하거나, 저장된 리소스를 갱신하는 데 사용해야 한다.
    - PUT 메서드는 "클라이언트가 기술한 URI로" 스토어에 새로운 리소스를 추가하는 데 사용한다.
    - PUT 요청 메세지에는 클라이언트가 저장하려는 리소스를 표현하는 부분이 BODY에 포함되어야 한다.
    - 예시 : 클라이언트 어플리케이션에서 어플리케이션 데이터를 서버에 저장할 수 있는 스토어 리소르를 제공하는 REST API
      - PUT /account/4ef2d5d0/buckets/objects/4321
  - PUT 메소드는 변경 가능한 리소스를 갱신하는데 사용해야 한다.

  - POST 메소드는 컬렉션에 새로운 리소스를 만드는 데 사용해야 한다.
    - 클라이언트는 POST 메소드를 사용 해 컬렉션 안에 새로운 리소스를 만든다.
    - POST 요청 바디에는, 새로운 리소스를 위해 제안된 상태 표현을 포함한다.
    - 예시 : 클라이언트가 POST를 사용해 컬렉션에 새로운 추가 사항을 요청
      - POST /leagues/seattle/teams/terbuchet/players
  - 또한, POST 메소드는 컨트롤러를 실행하는 데 사용해야 한다.

    - 클라이언트는 POST 메소드를 사용해 기능 지향적 컨트롤러 리소스를 동작시킨다.
    - 이때, 컨트롤러 메소드 기능의 입력 값을 헤더나 바디에 포함할 수 있다.
    - CRUD에 정확히 매핑되지 않는 경우에만, 컨트롤러 리소스를 사용할 수 있다!
      - 다시 말해, CRUD에 매핑된다면 다른 HTTP 메소드를 사용해야 한다!
    - 예시 : POST 요청 메소드를 사용하여 컨트롤러를 수행
      - POST /alerts/245743/resend

  - DELETE 메소드는 그 부모에서 리소스를 삭제하는 데 사용해야 한다.
    - 클라이언트는 DELETE 메소드를 통하여 컬렉션이나 스토어인 부모에서, 리소스를 완전히 제거한다.
    - 예시 : 클라이언트가 스토어에서 도큐먼트를 제거하는 예시
      - DELETE /accounts/4ef2d5d0/buckets/objects/4321
    - DELETE 메서드는 매우 명확한 의미로 사용되며, 오버로드되거나 확장될 수 없다.
      - 어떤 경우에도, 다른 기능에 DELETE 메소드를 사용하면 안 된다!
      - 예를 들어, 어떤 API에서 리소스를 완전히 지우지 않고, 임시 삭제/상태 변경 기능만을 제공하려면, 특별한 컨트롤러 리소스를 제공하고 DELETE 대신 POST 메소드를 사용해야 한다!
  - OPTIONS 메소드는 리소스의 사용 가능한 인터랙션을 기술한 메타데이터를 가져오는 데 사용해야 한다.
    - 클라이언트는 OPTONS 메소드를 사용하여, Allow 헤더에 포함된 리소스 메타데이터를 가져온다.
    - 예를 들어, 다음과 같다.
      - Allow : GET, PUT, DELETE

### 응답 상태 코드

- Status-Line (응답 상태의 첫 줄) 은 다음과 같다.

  - HTTP/1.1 403 Forbidden

- 응답 상태의 표준 범주는 다음과 같다.
  - 1xx : 정보, 전송 프로토콜 수준의 정보 교환
  - 2xx : 성공, 클라이언트 요청이 성공적으로 수행됨
  - 3xx : 재전송, 클라이언트는 요청을 완료하기 위해 추가적 행동을 취해야 함.
  - 4xx : 클라이언트 오류, 클라이언트의 잘못된 요청
  - 5xx : 서버 오류, 서버쪽 오류로 인한 상태 코드

### 2xx 상태 코드 규칙

- 여러 규칙들
  - 200("OK") 는 일반적인 요청 성공을 나타내는 데 사용해야 한다.
    - REST API가 성공적으로 수행되었음을 나타내는 코드!
    - 응답 바디가 포함된다.
  - 200("OK")는 응답 바디에 에러를 전송하는 데 사용해서는 안 된다.
    - 200("OK") , { "status" : "error" } 하지 말자..
  - 201("Created")는, 성공적으로 리소스를 생성했을 때 사용해야 한다.
    - 클라이언트 요청으로 새로운 리소스를 이용해 컬렉션에 생성했거나 스토어에 추가했을 때, 201 상태 코드로 응답한다.
    - 컨트롤러의 행동으로 새로운 리소스가 생겼을 경우에도, 201 상태 코드로 응답한다.
  - 202("Accepted")는 비동기 처리가 성공적으로 시작되었음을 알릴 때 사용해야 한다.
    - 202 응답은 클라이언트 요청이 비동기적으로 처리될 것임을 알려준다.
    - 요청은 유효했지만, 최종적으로 처리되기까진 시간이 걸릴 수 있단걸 알려준다.
  - 204("No Content") 는 응답 바디에 의도적으로 아무것도 포함하지 않을 때 사용한다.
    - 보통 PUT, POST, DELETE 요청에 대한 응답으로 사용한다.
    - GET의 경우에도 204로 응답할 수 있다!
      - 리소스는 존재하나, 바디에 포함시킬 어떠한 상태 표현도 가지지 않다는 것을 나타낼 수 있다.

### 3xx 상태 코드 규칙

- 여러 규칙들
  - 301("Moved Permanently") 는 리소스를 이동시켰을 때 사용한다.
    - REST API 리소스 모델이 상당 부분 재설계되었거나, 계속 사용할 새로운 URI를 클라이언트가 요청할 리소스에 할당하였다는 것을 나타낸다.
    - 응답의 Location 헤더에 새로운 URI를 기술해야 한다.
  - 302("Found") 는 사용하지 않는다.
    - 302 대신 303("See Other"), 307("Temporary Redirect") 사용하자.
  - 303("See Other")은 다른 URI를 참고하라고 알려줄 때 사용한다.
    - 처리가 끝난 컨트롤러 리소스가, 원하지 않는 응답 바디 대신 응답 리소스의 URI를 보냈음을 나타낸다.
    - 임시 상태 메시지의 URI일수도, 존재하는 리소스의 URI일수도 있다.
    - 클라이언트는 응답 Location 헤더에 있는 값으로 GET 요청 보낼 수 있다!
  - 304("Not Modified")는 대역폭을 절약할 때 사용한다.
    - 304 응답 코드는 Body에 아무것도 없다.
    - 204와 유사하지만..
      - 204는 Body에 보낼 내용이 없을 때 사용하지만
      - 304는 리소스에 대한 상태 정보가 있긴 하지만, 클라이언트에 이미 해당 상태의 최신 버전이 있음을 의미할 때 사용한다.
    - 조건부 HTTP 요청에 사용!
  - 307("Temporary Redirect")는, 클라이언트가 다른 URI로 요청을 다시 보내게 할 때 사용한다.
    - REST API가 클라이언트 요청을 처리하지 않을 것임을 나타낸다.
    - 클라이언트는 응답 메세지의 Location 헤더에 기술된 URI로 요청을 다시 보내야 한다.

### 4xx 상태 코드 규칙

- 여러 규칙들

  - 4xx번대 범주의 오류는, 클라이언트 오류를 기술하는 도큐먼트를 응답 Body에 포함할 수 있다!
  - 400("Bad Request")는 일반적인 요청 실패에 사용한다.
    - 다른 적절한 4xx 오류 값이 없을 때 사용하는 일반적 클라이언트 에러 상태이다.
  - 401("Unauthorized")는 클라이언트 인증에 문제가 있을 때 사용해야 한다.
    - 적절한 인증 없이 보호된 리소스를 사용하려 할 때 발생한다!
  - 403("Forbidden")은 인증 상태에 상관없이 액세스를 금지할 때 사용한다.
    - 클라이언트 요청은 정상이지만, REST API가 요청에 응하지 않는 경우를 나타낸다.
    - 즉, 클라이언트 인증에 문제가 있어서 발생하는 것이 아니다.
    - 예를 들어, 클라이언트가 REST API 리소스의 전체가 아니라 일부에 대한 접근만 허가된 경우가 있다.
    - 클라이언트가 허용된 범위 외의 리소스 접근 시 403으로 응답해야 한다.
  - 404("Not Found")는 요청 URI에 해당하는 리소스가 없을 때 사용해야 한다.
    - 말 그대로, 요청한 URI에 해당하는 리소스가 없을 때 사용한다.
  - 405("Method Not Allowed")는 HTTP 메소드가 지원되지 않을 때 사용해야 한다.
    - 클라이언트가 허용되지 않은 HTTP 메서드를 사용하려 할 때, 405 오류 응답을 한다.
    - 예를 들어 읽기 전용 리소스는 GET, HEAD 메서드만 지원한다.
    - 405 응답에는 Allow 헤더가 포함되어야 하며, 그 값으로 리소스가 지원하는 HTTP 메소드를 다음과 같이 나타내야 한다.
      - Allow : GET, POST
  - 406("Not Acceptable")은 요청된 리소스 미디어 타입을 제공하지 못 할 때 사용해야 한다.
    - 406 응답은 클라이언트의 Accept 요청 헤더에 있는 미디어 타입 중 해당하는 것을 만들지 못할 때 발생한다.
    - 예를 들어, 클라이언트가 XML 포맷 데이터를 요청했는데 API가 json밖에 제공하지 못 한다면 406 응답을 보낸다.
  - 409("Conflict")는 리소스 상태에 위반되는 행위를 했을 때 사용해야 한다.
    - 예를 들어, 클라이언트가 비어 있지 않은 스토어 리소스를 삭제하라고 요청하면 409 응답 오류를 보낸다.
  - 412("Precondition Failed")는 조건부 연산을 지원할 때 사용한다.
    - 특정한 조건이 만족될 때만 요청이 수행되도록 REST API로 알려준다.
    - 클라이언트가 요청 헤더에 하나 이상의 전제 조건을 지정할 경우 발생한다.
    - 이 조건들이 만족되지 않으면, 요청 수행 대신 412 응답을 보낸다.
  - 415("Unsupported Media Type")은 요청의 페이로드에 있는 미디어 타입이 처리되지 못 했을 때 사용해야 한다.
    - 요청 헤더의 Content-Type에 기술한 클라이언트가 제공한 미디어 타입을 처리하지 못 할 때 발생한다.
    - 예를 들어, 클라이언트가 XML을 보냈는데 API가 JSON밖에 처리 못 하면 415 응답 한다!
    - 406은 클라이언트가 요청한 포맷을 만들지 못할 경우, 415는 클라이언트가 보낸 데이터를 처리하지 못 하는 경우 사용한다.

  ### 5xx 상태 코드 규칙

  - 500("Internal Server Error")는 API가 잘못 작동할 때 사용해야 한다.
    - 서버에 문제가 있을 때 사용!
