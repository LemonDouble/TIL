# HTTP : HTTP 메서드 - 활용

분류: HTTP
작성일시: 2021년 7월 20일 오후 3:41

## 1. 클라이언트에서 서버로 데이터 전송

- 데이터 전달 방식은 크게 2가지
    1. Query 파라미터를 통한 데이터 전송
        - GET
        - 주로 정렬 필터(검색어)
            - 검색할때 KEY를 넣거나 (google 검색창 검색 내용 저장)
            - 게시판에 정렬 조건 넣거나 (인기순, 조회순..)
    2. Message Body를 통한 데이터 전송
        - POST, PUT, PATCH
        - 회원 가입, 상품 주문, 리소스 등록, 리소스 변경

- 클라이언트 → 서버의 4가지 상황
    - 정적 데이터 조회
        - 이미지, 정적 텍스트 문서
        - 조회의 경우, GET 사용
        - 예시) start.jpg를 받아오는 경우

        ```java
        //Request
        GET /static/star.jpg HTTP/1.1
        Host: www.aaa.com

        //Response
        HTTP/1.1 200 OK
        Content-Type: image/jpeg
        Content-Length: 34012

        12i309u12j312k30192u3123jo12u3128312...
        ```

    - 동적 데이터 조회
        - 검색, 게시판 목록에서 정렬 필터(검색어)
        - 조회의 경우, GET 사용
        - Query Parameter 사용해 데이터를 전달
        - 예시) Google에 hello를 검색하는 경우

        ```java
        //Request
        GET /search?q=hello&hl=ko HTTP/1.1
        Host: www.google.com

        //Response
        //동적으로 Data 생성 후 반환
        ```

    - HTML Form을 통한 데이터 전송
        - 회원 가입, 상품 주문, 데이터 변경
        - 예시) Form을 사용한 데이터 전송

        ```java
        //다음과 같은 HTML form의 username에 kim, age 란에 20 적었을 때
        <form action="/save" method="post">
        	<input type="text" name="username" />
        	<input type="text" name="age" />
        	<button type="submit">전송</button>
        </form>

        //Request HTTP Message
        POST /save HTTP/1.1
        Host: www.aaa.com
        //application/x-www-form-urlencoded : form, 약속된 규격!
        //url encoding 처리해서, abc김 -> abc%EA%B9%80 식으로 처리한다.
        Content-Type: application/x-www-form-urlencoded

        //BODY에 데이터 담겨서 전송된다
        usename=kim&age=20

        //Response
        //서버에서 데이터 저장
        ```

        - 파일 전송시 (multipart/form-data)
        - 위 Form에서, File 추가해서 보낼 때

        ```java
        //Binary 데이터를 보내기 위해 multipart/form-data 선언해줘야 한다
        <form action="/save" method="post" enctype="multipart/form-data">
        	<input type="text" name="username" />
        	<input type="text" name="age" />
        	//Form에 파일 추가, intro.png 업로드 했다고 가정
        	<input type="file" name="file1" />
        	<button type="submit">전송</button>
        </form>

        //Request HTTP Message
        POST /save HTTP/1.1
        Host: www.aaa.com
        //multipart-form data!
        Content-Type: multipart/form-data; boundary=----XXX
        Content-Length : 10457

        -----XXX
        Content-Disposition: form-data; name="username"

        kim
        -----XXX
        Content-Disposition: form-data; name="age"

        20
        -----XXX
        Content-Disposition: form-data; name="file1"; filename="intro.png"
        Content-Type: image/png

        10123afd78123bjk123y7981fga127....
        -----XXX--
        ```

    - HTTP API를 통한 데이터 전송
        - 회원 가입, 상품 주문, 데이터 변경
        - 서버 to 서버
            - 백엔드 시스템 통신
        - 앱 클라이언트
            - 아이폰, 안드로이드
        - 웹 클라이언트(Ajax)
            - HTML에서 Form 대신 JavaScript 통한 통신에서 사용
            - React, VueJs 같은 웹 클라이언트와 API 통신
        - POST, PUT, PATCH : 메세지 바디를 통해 데이터 전송
        - GET : 조회 ,쿼리 파라미터로 데이터 전달
        - Content-Type : application/json을 주로 사용 (사실상의 표준)
            - TEXT, XML, JSON등 있지만, JSON이 사실상의 표준이다.

        - 직접 HTTP 메세지 만들어서 보내면 된다!

        ```java
        //Request
        POST /members HTTP/1.1
        Content-Type: application/json

        {
        	"age": 20
        }
        ```

## 2. HTTP API 설계 예시

- HTTP API - 컬렉션 (많은 경우 컬렉션 사용)
    - POST 기반 등록
    - 예시 ) 회원 관리 API 제공

- API 기본 설계
    - 회원 목록 조회 : GET /members
    - 회원 등록 : POST /members
    - 회원 조회 : GET /members/{id}
    - 회원 수정 : PATCH,PUT,POST /members/{id}
        - PATCH : 개념적으로 가장 좋다. (부분 수정)
        - PUT : Clinet에서 완전히 덮어버려도 되는 경우 (예시 : 게시글 수정)
    - 회원 삭제 : DELETE /members/{id}

- POST 기반 등록 시스템 제작시
    - 클라이언트는 등록될 리소스의 URL를 모른다.
        - 회원 등록 : POST /members
    - 서버가 새로 등록되는 리소스 URI를 생성해 ㅈ준다.
        - HTTP 1.1 201 Created

            Location : /members/100 << 

    - 컬렉션 (Collection)
        - 서버가 관리하는 리소스 디렉토리
        - 서버가 리소스의 URI를 생성하고 관리
        - 여기서 컬렉션은 /members

---

- HTTP API - 스토어
    - PUT 기반 등록
    - 예) 정적 컨텐츠 관리, 원격 파일 관리

- PUT 기반 파일 관리 시스템 제작시 API 기본 설계
    - 파일 목록 : GET /files
    - 파일 조회 : GET /files/{filename}
    - 파일 등록 : PUT /files/{filename}
        - 기존 파일 있다면 : 지우고 새 파일로 덮어버린다
        - 기존 파일 없다면 : 새로운 파일을 업로드한다.
    - 파일 삭제 : DELETE /files/{filename}
    - 파일 대량 등록 : POST /files

- PUT 기반 등록 시스템 제작시
    - 클라이언트가 리소스 URI를 알고 있어야 한다.
        - 파일 등록 : PUT /files/{filename} 을 Client가 알고 있어야 한다.
    - 클라이언트가 직접 리소스의 URI를 지정한다.
    - 스토어 (Store)
        - 클라이언트가 관리하는 리소스 저장소
        - 클라이언트가 리소스의 URI를 생성하고 관리
        - 여기서 컬렉션은 /members

---

- HTML FORM 사용
    - 웹 페이지 회원 관리
    - HTML Form은 GET, POST만 지원
    - AJAX 이용하면 해결할 수 있지만, 순수 HTML FORM만 사용하는 경우

- HTML Form 사용시, API 기본 설계
    - 회원 목록 : GET /members
    - 회원 등록 폼 : GET /members/new
    - 회원 등록 : POST /members/new or POST /members
        - 가능하면, 회원 등록 폼과 처리 URL을 맞추자 (POST /members/new 사용)
        - 새로고침 눌렀을 때 쉽게 회원 등록 폼으로 돌아갈 수 있다.
        - 사람마다 취향 차이는 있을 수 있다!
    - 회원 조회 : GET /members/{id}
    - 회원 수정 폼 : GET /members/{id}/edit
    - 회원 수정 : POST /members/{id}/edit or POST /members/{id}
    - 회원 삭제 : POST /members/{id}/delete

- 컨트롤 URI
    - 위의 /edit, /delete 등...
    - GET, POST만 지원하므로 제약이 있음
    - 이런 제약을 해결하기 위해 "동사"로 된 리소스 경로 사용
    - HTTP 메서드로 해결하기 애매한 경우 사용

- 최대한 Resource 기반으로 설계하되, 정말 안 되는 경우만 컨트롤 URI를 대체제로 사용하자

## 3. 참고하면 좋은 URI 설계 개념

- 문서(Document)
    - 단일 개념( 파일 하나, 객체 인스턴스, 데이터베이스 Row)
    - 예) /members/100, /files/satr.jpg

- 컬렉션(collection)
    - 서버가 관리하는 리소스 디렉터리
    - 서버가 리소스의 URI를 생성하고 관리
    - 예) /members

- 스토어(store)
    - 클라이언트가 관리하는 자원 저장소
    - 클라이언트가 리소스의 URI를 알고 관리
    - 예) /files

- 컨트롤러(controller), 컨트롤 URI
    - Document, Collection, Store로 해결하기 힘든 추가 프로세스 실행
    - 동사를 직접 사용
    - 예) /members/{id}/delete

- 실제로 복잡한 설계 환경에서는 Control URI가 필요한 경우가 정말 많다!
- Document, Collection, Store을 최대한 많이 사용하고, 정말 안 되는 경우 Control URI 사용하자

- https://restfulapi.net/resource-naming