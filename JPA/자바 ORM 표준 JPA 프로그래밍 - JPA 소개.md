# 자바 ORM 표준 JPA 프로그래밍 - JPA 소개

분류:  JPA
자료 링크: https://www.inflearn.com/course/ORM-JPA-Basic/dashboard
작성일시: 2021년 7월 22일 오후 5:42

## 1. SQL 중심적인 개발의 문제점

- 어플리케이션은 객체 지향 언어 (Java, Scala..)
- 데이터베이스는 관계형 DB (Oracle, MySQL...)

- 따라서, 현대 개발에서 객체 → 관계형 DB 저장이 중요하다
    - 이러기 위해서는 SQL 작성이 필수적! (RDB는 SQL만 알아들으므로)

- SQL 중심 개발의 문제점
    - CRUD, Java 객체 → SQL / SQL → Java 객체 등은 중복적인 작업이다!
    - 필드가 추가되는 경우.. 중복된 쿼리 반복해서 작성해야 함

    ![%E1%84%8C%E1%85%A1%E1%84%87%E1%85%A1%20ORM%20%E1%84%91%E1%85%AD%E1%84%8C%E1%85%AE%E1%86%AB%20JPA%20%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%80%E1%85%B3%E1%84%85%E1%85%A2%E1%84%86%E1%85%B5%E1%86%BC%20-%20JPA%20%E1%84%89%E1%85%A9%E1%84%80%E1%85%A2%20bedc5450a97543f09086c845a2b84542/Untitled.png](https://github.com/LemonDouble/TIL/blob/main/JPA/img/Untitled.png)

    - 패러다임의 불일치
        - OOP : 추상화, 캡슐화, 정보은닉, 상속, 다형성 등 시스템의 복잡성을 제어하는데 초점
        - RDB : 데이터를 정교화시켜 잘 저장하는데 초점

- 객체와 관계형 RDB의 차이
    1. 상속
        - RDB에는 상속이 없다!

            ![%E1%84%8C%E1%85%A1%E1%84%87%E1%85%A1%20ORM%20%E1%84%91%E1%85%AD%E1%84%8C%E1%85%AE%E1%86%AB%20JPA%20%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%80%E1%85%B3%E1%84%85%E1%85%A2%E1%84%86%E1%85%B5%E1%86%BC%20-%20JPA%20%E1%84%89%E1%85%A9%E1%84%80%E1%85%A2%20bedc5450a97543f09086c845a2b84542/Untitled%201.png](https://github.com/LemonDouble/TIL/blob/main/JPA/img/Untitled%201.png)

            - 슈퍼타입, 서브타입이 상속과 비슷하긴 하다.
                - 하지만 Java 코드와는 다르게, 만약 저장하려면 Insert문 2개 (ITEM, ALBUM) 필요..
                - 조회하려면 JOIN 만들고 각각 객체 만들고.... 굉장히 복잡하다.
    2. 연관관계
        - 객체 : Reference를 통해 객체를 찾아온다.
        - RDB : PK, FK 이용하여 데이터를 찾는다.

        ![%E1%84%8C%E1%85%A1%E1%84%87%E1%85%A1%20ORM%20%E1%84%91%E1%85%AD%E1%84%8C%E1%85%AE%E1%86%AB%20JPA%20%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%80%E1%85%B3%E1%84%85%E1%85%A2%E1%84%86%E1%85%B5%E1%86%BC%20-%20JPA%20%E1%84%89%E1%85%A9%E1%84%80%E1%85%A2%20bedc5450a97543f09086c845a2b84542/Untitled%202.png](https://github.com/LemonDouble/TIL/blob/main/JPA/img/Untitled%202.png)

        - 객체는 Member → Team 이동은 가능하지만, Team → Member 이동은 불가능하다.
        - RDB에서는 FK 이용하여 Member→Team, Team → Member 양방향 이동이 가능하다.

        ![%E1%84%8C%E1%85%A1%E1%84%87%E1%85%A1%20ORM%20%E1%84%91%E1%85%AD%E1%84%8C%E1%85%AE%E1%86%AB%20JPA%20%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%80%E1%85%B3%E1%84%85%E1%85%A2%E1%84%86%E1%85%B5%E1%86%BC%20-%20JPA%20%E1%84%89%E1%85%A9%E1%84%80%E1%85%A2%20bedc5450a97543f09086c845a2b84542/Untitled%203.png](https://github.com/LemonDouble/TIL/blob/main/JPA/img/Untitled%203.png)

    3. 데이터 타입

    ![%E1%84%8C%E1%85%A1%E1%84%87%E1%85%A1%20ORM%20%E1%84%91%E1%85%AD%E1%84%8C%E1%85%AE%E1%86%AB%20JPA%20%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%80%E1%85%B3%E1%84%85%E1%85%A2%E1%84%86%E1%85%B5%E1%86%BC%20-%20JPA%20%E1%84%89%E1%85%A9%E1%84%80%E1%85%A2%20bedc5450a97543f09086c845a2b84542/Untitled%204.png](https://github.com/LemonDouble/TIL/blob/main/JPA/img/Untitled%204.png)

    1. 데이터 식별 방법

- 정리 : RDB는 자바의 Object와 다르다!
- 따라서, OOP적 설계에서의 장애가 된다!

## 2. JPA 소개

- JPA : Java Persistence API
    - 자바 진영의 ORM 기술 표준

- ORM : Object-Relational Mapping (객체-관계 매핑)
    - 객체는 객체대로 설계
    - RDB는 RDB대로 설계
    - ORM 프레임워크가 중간에서 매핑
    - 대중적인 언어에는 대부분 ORM 기술이 존재

- JAVA 어플리케이션 → JPA → JDBC API → SQL → DB

- JPA의 역할
    - 데이터 삽입시
        - Entity 분석
        - Insert SQL 생성
        - JDBC API 사용
        - (중요) 패러다임 불일치 해결

    - 데이터 조회시
        - Select SQL 생성
        - JDBC API 사용
        - ResultSet 매핑
        - 패러다임 불일치 해결

- JPA는 표준 명세이다.
    - JPA는 인터페이스의 모음!
    - JPA 2.1 표준 명세를 구현한 구현체 따로 있다
    - 하이버네이트, EclipseLink, DataNucleus

- JPA를 왜 사용해야 하는가?
    - SQL 중심적 개발에서, 객체 중심적으로 개발
    - 생산성
        - 저장 : jpa.persist(member)
        - 조회 : Member member = jpa.find(memberId)
        - 수정 : member.setName("변경할 이름")
        - 삭제 : jpa.remove(member)
    - 유지보수
        - 만약 필드가 추가되어도 쿼리 한땀한땀 다 수정 안 해도 된다!
    - 패러다임 불일치 해결
        - 상속
            - 개발자는 단순히 jpa.persist(album);
            - JPA는 INSERT ITEM, INSERT ALBUM 만들어 준다!
        - 조회
            - 개발자는 단순히 Album album = jpa.find(Album.class, albumId);
            - JPA는 SELECT I.*, A.* FROM ITEM i Join ALBUM A ON I.ITEM-iD = A.ITEM-ID
            - 조인도 알아서 해 준다!
        - 연관관계, 객체 그래프 탐색
            - 다음과 같이, 마치 Java.collections 에 넣고 뺴는것처럼 사용이 가능하다!

                ```java
                member.setTeam(team);
                jpa.persist(member);

                Member member = jpa.find(Member.class, memberId);
                Team team = member.getTeam();
                ```

            - 아래 레이어 (DAO : 데이터 계층) 에 대한 신뢰가 가능하다!

                ```java
                Sring memberId = 100;
                Member member1 = jpa.find(Member.class, memberId);
                Member member2 = jpa.find(Member.class, memberId);

                member1 == member2 //같다!
                ```

            - JPA는, 같은 트랜잭션에서 조회한 Entity는 같은 Entity임을 보장한다.
    - 성능
        - 1차 캐시와 동일성 (identity)
            - 같은 트랙제션 안에서는 같은 엔티티를 반환 - 아주 약간의 조회 성능 향상

                ```java
                Sring memberId = 100;
                Member member1 = jpa.find(Member.class, memberId); //SQL
                Member member2 = jpa.find(Member.class, memberId); //CACHE

                member1 == member2 //true
                ```

            - DB Isolation Level이 Read Commit이어도, 애플리케이션에서 Repeatable Read 보장 (Application Layer에서 뭔가 해 주니까 DB의 Isolation Level을 한단계 내려도 되는구나 정도..)
        - 트랜잭션을 지원하는 쓰기 지연(transactional write-behind)
            - Transaction을 커밋할 때까지 Insert SQL을 모음
            - JDBC BATCH SQL 기능을 사용해 한번에 SQL 전송

                ```java
                transaction.begin(); //트랜잭션 시작

                em.persist(memberA);
                em.persist(memberB);
                em.persist(memberC);
                // 여기까지 INSERT SQL을 데이터베이스에 보내지 않는다.

                //커밋하는 순간 데이터베이스에 INSERT SQL을 모아서 보낸다.
                transaction.commit(); //트랜잭션 커밋
                ```

        - 지연 로딩 (Lazy Loading)과 즉시 로딩
            - 지연 로딩 : 객체가 실제 사용될 때 로딩

                ```java
                Member member = memberDAO.find(memberId); //SELECT * FROM MEMBER
                Team team = member.getTeam();
                String teamName = team.getName(); //SELECT * FROM TEAM

                //문제 : Query
                ```

            - 즉시 로딩 : JOIN SQL로 한번에 연관된 객체까지 미리 조회

                ```java
                Member member = memberDAO.find(memberId);
                // SELECT M.*, T.* FROM MEMBER JOIN TEAM ...
                Team team = member.getTeam();
                String teamName = team.getName();
                ```

            - 만약, Member 조회하는 경우 (거의 항상) Team도 같이 로딩 → 즉시 로딩
            - Member을 조회하고 Team을 잘 조회하지 않음 → 지연 로딩
            - JPA는 옵션으로 두 모드를 키고 끌 수 있다!
    - 데이터 접근, 추상화와 벤더 독립성
    - 표준
