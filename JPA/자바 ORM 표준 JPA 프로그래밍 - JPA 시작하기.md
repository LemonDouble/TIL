# 자바 ORM 표준 JPA 프로그래밍 - JPA 시작하기 (setting)

분류:  JPA
작성일시: 2021년 7월 22일 오후 8:39
수정일시 : 2021년 7월 26일 - persistence.xml, 하이버네이트 버전 수정

## 1. 프로젝트 생성 및 설정

- H2 Database 설치
    - 웹용 쿼리 툴 제공
    - MYSQL, Oracle DB 시뮬레이션 기능
    - 시퀀스, Auto Increment 기능 지원

- Maven Project 생성
    - pom.xml 설정

    ```xml
   
    ```

    - Jpa 설정
        - src/main/resource/META-INF/persistence.xml (파일 위치는 고정이다!)

        ```xml
        ```

- 데이터베이스 방언
    - JPA는 특정 데이터베이스에 종속되지 않는다!
    - 각각의 DB가 제공하는 SQL 문법과 함수는 조금씩 다르다.
        - 가변문자 : MySQL은 VARCHAR, Oracle은 VARCHAR2
        - 문자열을 자르는 함수 : SQL 표준은 SUBSTRING(), Oracle은 SUBSTR()
        - 페이징 : MySQL은 LIMIT, Oracle은 ROWNUM

    - 방언 : SQL을 표준을 지키지 않는 특정 데이터베이스만의 고유한 기능
    - hibernate.dialect의 속성으로 DB 방언을 설정할 수 있다 (persistence.xml)
        - H2 : org.hibernate.dialect.H2Dialect
        - Oracle 10g : org.hibernate.dialect.Oracle10gDialect
        - MySQL : org.hibernate.dialect.MySQL5InnoDBDialect

## 2. Hello JPA! 어플리케이션 개발

- JPA는 Persistence라는 Class에서 시작
    1. persistence.xml에서 설정 정보 읽음
    2. EntityManagerFactory라는 Class 만듬
    3. 이후 필요할떄마다, Entity라는 Class 만듬

- 객체와 테이블을 생성하고 매핑하기
    - @Entity : JPA가 관리할 객체
    - @Id : 데이터베이스의 PK와 매칭

    - RDB에서 테이블 생성!

        ```java
        ```

    - Java에서 Class 생성!

        ```java
        
        ```

- Main Class에서 다음과 같이 설정해서 DB에 INSERT 할 수 있다.

    ```java
    
    ```

- 조회의 경우

    ```java
    
    ```

- 삭제의 경우

    ```java
    
    ```

- 수정의 경우

    ```java
    
    ```

- 주의
    - EntityManagerFactory는 단 하나만 생성해서 어플리케이션 전체에서 공유
    - EntityManager는 Thread간 공유하면 안 된다. (사용하고 버려야 한다.)
    - JPA의 모든 데이터 변경은 Transaction 안에서 실행!

- JPQL 소개
    - JPA에서 복잡한 쿼리작성을 도와주는 방법
    - JPA를 사용하면, 엔티티 객체를 중심으로 개발하게 된다.
    - 하지만 검색 쿼리 등이 문제다!
    - 필요한 데이터만 DB에서 불러오려면, 결국 검색 조건이 포함된 SQL이 필요하다.
    - JPQL은 검색을 물리적 대상인 Table이 아닌, 엔티티 객체로 변환해서 쿼리를 만들 수 있다.
    - 이렇게 하면, 한단계 더 추상화된 에티티 객체를 대상으로 쿼리를 날리므로 설정값 변경 등으로 DB를 바꾸거나 할 수 있다. (데이터베이스 방언을 기억하자)
