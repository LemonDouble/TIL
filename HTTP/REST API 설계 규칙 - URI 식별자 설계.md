# REST API 설계 규칙 - URI 식별자 설계

분류: HTTP
작성일시: 2021년 9월 16일 오후 12:32

### URI ( Uniform Resource Identifier)

- REST API에서 리소스를 나타내는 방법
- http://api.example.com/france/paries/louvre/..
- 대상을 나타내는 역할을 맡는다!

### URI 형태

- 여러 규칙들
    - 슬래시 구분자(/) 는 계층 관계를 나타내는데 사용한다.
        - [http://api.example.com/](http://api.example.com/)shapes/polygons/quadrilaterals/squares

    - URI 마지막 문자로 슬래시를 포함하지 않는다.
        - [http://api.example.com/shapes](http://api.example.com/shapes)/ (x)
        - [http://api.example.com/shapes](http://api.example.com/shapes)/ (o)

    - 하이픈(-) 은 URI 가독성을 높이는 데 사용한다.
        - 공백을 하이픈으로 바꿀 수 있다!
        - http://api.example.com/blogs/mark-masse/entries/this-is-my-first-post

    - 밑줄(_)은 URI에 사용하지 않는다.
        - 밑줄 대신 하이픈을 사용하자!

    - URI 경로에는 소문자가 적합하다.
        - [http://api.example.com](http://api.example.com./)/my-foler (o)
        - [HTTP://API.EXAMPLE.COM/MY-FOL](http://api.EXAMPLE.COM/MY-FOLER)DER (x) ,
            - 하지만 URI 포맷 스펙에서(RFC 3986) 1과 같다고 간주한다.
        - [http://api.example.com/My-Fol](http://api.example.com/My-Foler)der (xxxx)
            - 위 URI와는 다른 URI이다. 대소문자를 섞어 사용하지 말자..

    - 파일 확장자는 URI에 포함시키지 않는다.
        - [http://api.example.com/student/123123/score.json](http://api.example.com/student/123123/score.json) (x)
        - [http://api.example.com/student/123123/score](http://api.example.com/student/123123/score.json) (o)
        - 파일 확장자를 포맷을 나타내는 용도로 사용해선 안 된다.
        - HTTP Header의 미디어 타입 협상을 사용하도록 하자.

### URI 권한 설계

- 여러 규칙들
    - API에 있어 서브 도메인은 일관성 있게 사용해야 한다.
        - API 최상위 도메인과 1차 서브 도메인 이름으로(soccer.restapi.com) 서비스 제공자를 식별해야 한다.
        - http://api.soccer.restapi.com

    - 클라이언트 개발자 포탈 서브 도메인 이름은 일관성 있게 만들어야 한다.
        - 보통 Rest API는 개발자 포탈(Developer Portal)을 지원한다.
        - 관습적으로 developer라는 서브 도메인을 사용한다.
        - http://developer.soccer.restapi.org

### 리소스 모델링

- http://api.soccer.restapi.org/leagues/seattle/teams/trebuchet
    - 이 URI 디자인은, 다음과 같은 자체 리소스 주소를 가진 URI도 존재함을 뜻한다.
    - http://api.soccer.restapi.org/leagues/seattle/teams
    - http://api.soccer.restapi.org/leagues/seattle
    - http://api.soccer.restapi.org/leagues
    - http://api.soccer.restapi.org

### 리소스 원형

- API Resource를 모델링 할 때, 기본적 원형 몇 개를 사용할 수 있다.
- 설계자는 이 원형을 통해, 디자인 패턴에서와 같이 REST API 디자인에서 보편적으로 쓰이는 리소스의 구조/행동을 일관된 방식으로 다룰 수 있다.

- Document
    - 객체 인스턴스/데이터베이스 레코드와 유사한 단일 개념
    - 도큐먼트의 상태는 일반적으로 값을 가진 필드, 다른 관련 리소스와의 링크를 가짐

    - 다음 URI는 각각 도큐먼트 리소스를 나타냄
        - http://api.soccer.restapi.org/leagues/seattle
        - http://api.soccer.restapi.org/leagues/seattle/teams/trebuchet
        - http://api.soccer.restapi.org/leagues/seattle/teams/trebuchet/players/mike

    - 논리적으로 도큐먼트는 docroot으로 알려진 REST API 루트 리소스 후보에 해당한다.
    - 이때 docroot는 REST API의 공개된 진입점과 같다.
        - http://api.soccer.restapi.org

### 컬렉션

- 컬렉션 리소스는 서버에서 관리하는 디렉터리라는 리소스
- 클라이언트는 새로운 리소스를 제안, 컬렉션에 포함시킬 수 있음
- 하지만, 새로운 리소스를 생성할지는 컬렉션이 결정

- 다음 URI는 각각 컬렉션 리소스를 나타냄
    - http://api.soccer.restapi.org/leagues
    - http://api.soccer.restapi.org/leagues/seattle/teams
    - http://api.soccer.restapi.org/leagues/seattle/teams/trebuchet/players

### 스토어

- 클라이언트에서 관리하는 리소스 저장소
- 스토어 리소스는 API 클라이언트가 리소스를 넣거나 빼는 것, 지우는 것에 관여
- 스토어 스스로 새로운 리소스 생성 불가, 새로운 URI 만드는 것도 불가하다.
- 대신 스토어 리소스는 최초 생성시, 클라이언트가 선택한 URI 가짐

- 클라이언트 프로그램에서 ID가 1234인 사용자를 보여주고, 가상의 Soccer REST API를 사용해 favorites라는 스토어에 alonso라는 도큐먼트 리소스를 넣는 예시
    - PUT /users/1234/favorites/alonso

### 컨트롤러

- 컨트롤러 리소스는 실행 가능한 함수와 같다.
    - 즉, 파라미터 (입력값) 과 반환 값(출력 값) 있다.
- CRUD와 논리적으로 매핑되지 않는 어플리케이션 고유의 행동을 컨트롤러 리소스의 도움을 받아 수행

- 클라이언트가 사용자에게 경고를 재전송하게 하는 컨트롤러 리소스
    - POST /alerts/245743/resend

---

### URI 경로 디자인

- 여러 규칙들
    - 도큐먼트 이름으로는 단수 명사를 사용해야 한다.
        - 예를 들어, 한 명의 선수 도큐먼트 URI는 단수 형태가 되어야 한다.
        - [http://api.soccer.com/](http://api.soccer.com/)leagues/seattle/teams/trebuchet/players/claudio
    - 컬렉션 이름으로는 복수 명사를 사용해야 한다.
        - 컬렉션을 식별하는 URI는 복수 명사나, 복수 명사구를 나타내는 명사로 경로의 이름을 지어야 한다.
        - 예를 들어, 선수 도큐먼트의 컬렉션 URI는 포함된 리소스의 복수 명사 형태로 나타낸다.
            - [http://api.soccer.com/](http://api.soccer.com/)leagues/seattle/teams/trebuchet/players
    - 스토어 이름으로는 복수 명사를 사용해야 한다.
        - 예를 들어서, 음악 플레이리스트 스토어의 URI는 다음과 같은 복수 명사 형태를 사용한다.
        - [http://api.mustic.com/](http://api.mustic.com/)artists/mikemassedotcom/playlists
    - 컨트롤러 이름으로는 동사나 동사구를 사용해야 한다.
        - 프로그램의 Function처럼, 컨트롤러는 동작을 포함하는 이름으로 지어야 한다.
        - 예시
            - [http://api.college.com](http://api.college.com)/student/morgan/register
            - [http://api.example.com/lists/4324/dedupe](http://api.example.com/lists/4324/dedupe) (dedupe : 중복 제거)
            - http://api.ognom.com/dbs/reindex
            - http://api.build.com/qa/nightly/runTestSuite
    - 경로 중, 변하는 부분은 유일한 값으로 대체한다.
        - [http://api.soccre.com/leagues/{leagueId](http://api.soccre.com/leagues/{leagueId})}/teams/{teamId}/players/{playerId}
        - 이 때, leagueId, teamId, playerId는 유일한 값이어야 한다.
        - REST API Client는, URI가 유의미한 리소스 식별자임을 고려해야 한다.
        - URI를 유일한 ID로 사용해야만, 기존 클라이언트에 영향을 미치지 않고 REST API의 백엔드 시스템을 개선할 수 있다.
    - CRUD 기능을 나타내는 것은 URI에 사용하지 않는다.
        - URI는 오로지 리소스를 식별하는 데에만 사용해야 한다.
        - CRUD는 HTTP Request Method를 통해 설계되어야 한다.

---

### URI Query 디자인

- [http://api.college.com/student/morgan/send-sms](http://api.college.com/student/morgan/send-sms)
- [http://api.college.com/student/morgan/send-sms](http://api.college.com/student/morgan/send-sms)?text=hello

- 위는 문자를 보내는 컨트롤러 리소스 URI
- 아래는 "hello" 라는 문자를 보내는 컨트롤러 리소스 URI

- 리소스 URI 전체는 HTTP Cache같은 네트워크 기반 중개자에게 아무런 의미가 없어야 한다.
    - 즉, Query 자체도 Cache 가능해야 한다!
- 다시 말해, URI의 Query 유무에 따라 Cache의 작동/기능이 바뀌어선 안 된다.
    - 특히, 요청 URI에 Query 있다고 Response Message가 Cache에서 제외되어선 안 된다.
    - Query가 아닌 HTTP Heade가 Cache의 중간 역할을 결정해야 한다.

- 여러 규칙들
    - URI Query 부분으로 컬렉션/스토어를 필터링 할 수 있다.
        - GET /users → 컬렉션에 있는 모든 사용자의 리스트
        - GET /users?role=admin → 컬렉션에 있는 사용자 중 role이 admin인 사용자의 리스트

    - URI Query는 컬렉션이나 스토어의 결과를 페이지로 구분하여 나타나는 데 사용해야 한다.
        - REST API는 Query 구성요소를 사용해 컬렉션/스토어의 값을 pageSize, pageStartIndex 같은 파라미터 값으로 페이지화한다.
            - pageSize는 응답에 반환되는 엘리먼트의 최댓값을 나타내는 데 사용한다.
            - pageStartIndex 파라미터는 응답에 반환되는 첫 번째 엘리먼트의 인덱스를 나타낸다.
        - GET /users?pageSize=25&pageStartIndex=50
            - 이는 50Page부터 시작해 최대 75Page까지만을 요청하는 것이다!
        - 만약 URI Query만으로 클라이언트의 페이지/필터링에 대응할 수 없다면, 특별한 컨트롤러를 생각해 볼 수 있다.
            - 예를 들어, POST /users/search 와 같은 Controller로 복잡한 입력을 처리할 수도 있다.