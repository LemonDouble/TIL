# HTTP : URI와 네트워크 요청 흐름

분류: HTTP
작성일시: 2021년 7월 19일 오후 10:41

## 1. URI

- URI : Uniform Resource Identifier ([https://www.ietf.org/rfc/rfc3986.txt](https://www.ietf.org/rfc/rfc3986.txt))

- URI는 로케이터 (locator), 이름 (name) 또는 둘 다 추가로 분류될 수 있다.

- URL : Resource Locator → 리소스의 위치
- URN : Resource Name → 리소스의 이름
- URI : URL + URN

```
         foo://example.com:8042/over/there?name=ferret#nose -> URL( Resource Locator)
         \_/   \______________/\_________/ \_________/ \__/
          |           |            |            |        |
       scheme     authority       path        query   fragment
          |   _____________________|__
         / \ /                        \
         urn:example:animal:ferret:nose -> URN (Resource Name)
```

- URI의 뜻
    - Uniform : 리소스를 식별하는 통합된 방식
    - Resource : 자원, URI로 식별할 수있는 모든 것
    - Identifier : 다른 항목과 구분하는데 필요한 정보

- 하지만 URN은, 이름만으로 실제 리소스를 찾을 수 있는 방법이 보편화되어있지 않음.
- 따라서 URL과 URI를 같은 의미로 이야기 할 수 있다.

- URL 전체 문법
    - scheme://[userinfo@]host[:port][/path][?query][#fragment]

    - scheme : 프로토콜, http, https, ftp 등
    - userInfo : URL에 사용자 정보를 포함해서 인증, 하지만 거의 사용하지 않음
    - host : 도메인명, 혹은 IP 주소
    - port : 생략 가능, 접속 포트
    - path : 리소스 경로 (path), 계층적 구조 /member/100..
    - query : key=value 형태, ?로 시작, &로 추가 가능. query parameter, query string 등으로도 부른다. ?q=hello&hl=ko
    - fragment : html 내부 북마크 등에 사용. 서버에 전송되는 정보는 아니다.

## 2. 웹브라우저 요청 흐름

- [https://www.google.com:443/search?q=hello&hl=ko](https://www.ggogle.com:443/search?q=hello&hl=ko) 검색하는 경우

1. google.com의 DNS를 조회하여 IP 200.200.200.2를 확인한다.
2. 웹 브라우저가 HTTP Request Message 생성한다.

    ```java
    GET /search?q=hello&hl=ko HTTP:/1.1
    Host: www.google.com
    ```

3. SOCKET 라이브러리 통해 전달한다.
    - TCP/IP 연결 (IP, PORT)
    - 데이터 전달
4. Payload에 HTTP Request Message를 싣고, TCP/IP 패킷을 생성한다.
5. Google 서버가 HTTP Request Message를 해석한다.
6. Google 서버가 HTTP Response Message를 생성한다.

```java
HTTP/1.1 200 OK
Content-Type:text/html;charset=UTF-8
Content-Length:3423 //전체 HTML 데이터의 길이

<html>
....
```

1. Response Message를 해석하여 브라우저가 HTML을 렌더링한다.