# REST API 설계 규칙 - 메타데이터 디자인

분류: HTTP
작성일시: 2021년 9월 17일 오후 8:17

### HTTP 헤더

- HTTP Request/Response에 포함된 Header 통해 여러 형태의 Metadata가 전달된다.

- 여러 규칙들
    - 규칙 : Content-Type을 사용해야 한다.
        - Request/Response Body에 있는 데이터 타입을 나타낸다.
        - 이 헤더의 값은 "미디어 타입" 으로 알려진 특별하게 정의된 문자열이다.
        - application/json, application/x-www-form-urlencoded..
        - Client/Server는 이 헤더 값을 참조하여 메시지 바디 처리 방법을 결정한다.
    - 규칙 : Content-Length를 사용해야 한다.
        - 바이트 단위로 Body의 크기를 나타낸다.
        - Client는 이 값을 이용해 Byte 크기를 올바로 읽었는지 알 수 있다.
        - HEAD 요청을 사용하여, 다운로드 받지 않고도 Body가 얼마나 큰지 알 수 있다.
    - 규칙 : Last-Modified는 Response에서만 사용해야 한다.
        - 이 헤더의 값은 타임스탬프로, 리소스의 표현 값이 바뀐 마지막 시간을 나타낸다.
        - Clinet와 Cache 중간자는 이 값을 이용하여 리소스 갱신 여부를 결정한다.
    - 규칙 : Etag는 Response에서 사용해야 한다.
        - Etag는 응답 엔티티에 포함된 표현 상태의 특정 버전을 나타내는 일련의 문자열이다.
        - 이 엔티티는 HTTP 메세지의 Payload, 메세지가 헤더+바디로 구성되어 있다.
        - 태그 값은 리소스의 상태에 따라 바뀔 수 있으며, 어떤 문자열이라도 될 수 있다.
        - 항상 GET Method 요청에 대한 응답으로 보내져야 한다!
    - 규칙 : Store는 조건부 PUT 요청을 지원해야 한다.
        - 스토어는 PUT을 통해, 리소스 추가/갱신을 모두 처리한다.
        - 따라서, PUT 요청이 들어왔을 때 추가/갱신 중 어떤 요청인지 알기 어렵다.
        - Clinet의 If-Unmodified-Since, If-Match Header를 통해 의도를 추론할 수 있다.
            - If-Unmodified-Since는 API에게 리소스가 특정 시간 이후에 바뀌지 않았을 경우에만 동작하도록 요구한다.
            - If-Math 는 엔티티 태그인데, Etag를 통해 위와 같은 동작을 수행한다.
        - 예시를 통해 좀 더 자세히 이해해 보자!
            - Client 1, 2는 REST API의 /object 스토어 리소스를 통해 데이터를 공유한다.
            - Client 1이 데이터를 저장하기 위해, /object/2113으로 PUT 요청을 보낸다.
            - REST API는 데이터를 저장하고, 응답으로 201("Created")를 보낸다.
            - Clinet 2가 데이터를 공유하기 위해, /objects/2113으로 PUT 요청을 보낸다.
            - REST API는 클라이언트의 의도를 알 수 없다. Client 1이 이미 저장한 정보가 있는데 해당 내용을 덮어쓸 것인지? 아니면 새로 추가하는 것인지 알 수 없다.
            - 따라서, REST API는 409("Conflict")를 Response 한다.
                - 덧붙여 오류에 대한 부가 정보를 Body에 제공한다.
            - Client 2가 저장된 데이터를 갱신하려고 할 때는 If-Matched 헤더를 포함해 재요청한다.
                - 만약 제공된 Etag가 현재 서버의 Etag와 다를 경우, REST API는 412("Precondition Failed") 오류 응답 코드를 보낸다.
                - 제공된 조건이 일치한다면, REST API는 저장된 리소스의 상태를 갱신하며 200("OK")나 204("No Comment") 응답을 줄 수 있다.
    - 규칙 : Location은 새로 생성된 리소스의 URI를 나타내는데 사용해야 한다.
        - Location 응답 헤더의 값은 클라이언트의 관심 범위에 있는 리소스를 식별하는 URI이다.
        - 컬렉션/스토어에 성공적으로 리소스를 생성했다는 응답이며, REST API는 Location 헤더를 포함해 새로 생성된 리소스의 URI를 나타내야 한다.
        - 202("Accepted") 응답 안에 있는 헤더 값은, 비동기 컨트롤러 리소스의 연산 상태를 클라이언트에 알려 주는 데 사용할 수도 있다!
    - 규칙 : Cache-Cotnrol, Expires, Date 응답 헤더는 캐시 사용을 권장하는 데 사용해야 한다.
        - 상태 표현을 제공할 때, Cache-Control 헤더와 초 단위의 max-age 값이 같은 갱신 주기를 함께 제공해야 한다.
        - 예시 : 60초 동안 Cache 사용이 가능함을 알리는 헤더
            - Cache-Control : max-age=60, must-revalidate
        - 기존의 HTTP 1.0 캐시를 지원하려면, REST API는 만료일을 나타내는 Expires 헤더를 포함해야 한다. 이 값은 Response 생성 시간 + 갱신 주기가 더해진 날짜다.
        - 또한, REST API는 시간으로 표시되는 Date 헤더를 포함해 API의 응답 시간을 나타낸다.
        - Client는 Date, Expires 헤더를 통해 갱신 주기를 계산할 수 있다.
            - 예시:
                - Date : Tue, 15 Nov 1994 08:12:31 GMT
                - Expires: Thu, 01 Dec 1994 16:00:00 GMT
    - 규칙 : Cache-Control, Expires, Pragma 응답 헤더는 캐시 사용을 중지하는데 사용해야 한다.
        - REST API의 응답을 캐시에 저장하지 않도록 하려면, Cache-Control 헤더 값을 no-cache나 no-store로 설정하면 된다.
        - 이 경우, HTTP 1.0 캐시와 상호 운용을 고려한다면
            - Pragma:no-cache와 Expires:0 헤더를 추가한다.
    - 규칙 : Cache 기능은 사용해야 한다.
        - no-cache 지시자는 캐시 응답이 제공되는 것을 막는다.
        - REST API는 꼭 필요한 경우가 아니면 캐시를 사용하지 않으므로, no-cache 지시자를 추가하는 대신 값이 작은 max-age를 사용하면, 클라이언트는 갱신에 관계없이 짧은 시간 내 캐시에 저장된 복사본을 가져올 수 있다.
    - 규칙 : Expire Cache 헤더는 200("OK") 응답에 사용해야 한다.
        - 성공적인 GET, HEAD 메소드 요청에 대한 응답으로만 Cache를 설정해야 한다!
        - 실제로 POST 메서드는 캐시에 저장 가능하지만, 대부분의 캐시는 이 메서드를 캐시에 저장하기 불가능한 것으로 취급한다. (중간 Cache)
        - 다른 메서드에 만기 헤더를 지정해선 안 된다!
    - 규칙 : Expire Cache 헤더는 "3xx", "4xx" 응답에 선택적으로 사될 수 있다.
        - "3xx", "4xx" 응답에 대한 Cache 헤더를 추가하는 방법을 "네거티브 캐싱" 이라고 한다.
        - Redirect 횟수, REST API 오류에 따른 부하를 감소시킬 수 있다.
    - 규칙 : Custom HTTP Header는 HTTP 메서드의 행동을 바꾸는 데 사용해서는 안 된다.
        - Custom Header는 정보 전달이 목적일 경우에만 선택적으로 사용할 수 있다.
        - Clinet/Server 둘 다 커스텀 헤더를 처리하지 못 하더라도, Client/Server 양 쪽에서 API를 처리하는데 문제가 없도록 구현해야 한다.
        - 만약 Custom HTTP Header로 전송하는 정보가 응답/요청을 적절하게 처리하는데 중요한 정보라면, 그 정보는 요청/응답 바디에 포함하거나, URI에 포함되어야 한다.

### 미디어 타입

- Request/Response Message Body 안에 있는 데이터 형태를 식별하기 위해, Content-Type Header를 사용한다.
- 안에 들어가는 값을 미디어 타입이라고 한다!

- type / subtype (; parameter)
    - type : application, audio, image, message, model, multipart, text, video
    - Content-type : text/html; charset=ISO-8859-4 와 같이 parameter 추가할 수 있다.

- 등록된 미디어 타입
    - 널리 사용하는 미디어 타입들을 설명한다!
    - text/plain
        - 특별한 콘텐츠/마크없 없는 평문 포맷
    - text/html
    - image/jpeg
    - application/xml
    - application/atom+xml
        - Feed로 알려진 구조적 데이터를 XML 기반의 리스트로 포맷팅한 Atom을 사용하는 컨텐츠
    - application/javascript
    - application/json

- 벤더 고유 미디어 타입
    - 서브 타입의 접두어로 "vnd" 사용
    - 특정 업체에서 소유/관리하고 있음을 나타낸다.
    - 예시
        - application/vnd.ms-excel
        - application/vnd.lotus-notes

### 미디어 타입 설계

(WRML에 대한 부분은 생략했음)

이유 : WRML을 실제로 사용하는 경우도 본 적이 없고, 검색해 봐도 실제로 광범위하게 사용되고 있는지 잘 모르겠다.. 덤으로 REST API를 오히려 더 복잡하게 만드는 것 같은 느낌도 들어서.. 

- 여러 규칙들
    - 규칙 : 리소스의 표현이 여러 가지 가능할 경우, 미디어 타입 협상을 지원해야 한다.
        - 클라이언트에서 원하는 미디어 타입을 Accept 헤더에 추가해, 특정 포맷/스키마를 협상할 수 있게 해야 한다.
        - 예시
            - Accept: application/json
    - 규칙 : Query 변수를 사용해 미디어 타입 선택을 지원할 수 있다.
        - 예시
            - GET /bookmarks/mike?accept=application/json
        - 이 방법은, URI Path에 .xml 등의 가상 파일 확장자를 추가하는 것 보다 명확하다!