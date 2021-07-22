# Spring Boot & JPA WebApp 개발 - 환경설정

분류: SPRING+JPA
자료 링크: https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-JPA-%ED%99%9C%EC%9A%A9-1/dashboard
작성일시: 2021년 7월 21일 오후 8:29

- Spring 공식 듀토리얼 : [https://spring.io/guides/gs/serving-web-content/](https://spring.io/guides/gs/serving-web-content/)

## 1. 프로젝트 생성

- Spring Boot Starter ( [https://start.spring.io/](https://start.spring.io/) )
    - web, thymeleaf, jpa, h2, lombok, validation

## 2. 라이브러리 의존관계 살펴보기

- gradle 의존관계 보기
- termial 에서 ./gradlew dependencies
- 혹은 Intelij에서 우측 Gradle → Dependencies

## 3. View 환경설정

- Thymeleaf : Markup을 깨지지 않으면서 데이터를 추가할 수 있다
- Markup을 깨지 않으므로, Template Engine 없이도 웹 브라우저로 확인할 수 있다

## 4. H2 Database 설치

- H2 DB 다운로드 → bin에서 [start.sh](http://start.sh) or start.bat 실행
- JDBC URL에 처음엔 jdbc:h2:~/프로젝트명 입력
- ~/프로젝트명.mv.db 파일 생긴것을 확인
- 이후 jdbc:h2:tcp://localhost/~/프로젝트명 으로 접근

## 5. JPA와 DB 설정, 동작 확인

- main/resources/application.yml

```java
spring:
  datasource:
    url: jdbc:h2:tcp://localhost/~/jpashop
    username: sa
    password:
    driver-class-name: org.h2.Driver

  jpa:
    hibernate:
      ddl-auto: create #Table을 만들어 주는 모드
    properties:
      hibernate:
#        show_sql: true #System.out을 통해 출력
        format_sql: true

logging:
  level:
    org.hibernate.SQL: debug # Logger 통해 출력
    org.hibernate.type: trace # 실제로 어떤 Data 들어갔는지 확인 binding parameter [1] as [VARCHAR] - [memberA]..
```

- Junit5만 사용할거라면 Build.gradle에서 Junit4 Depenencies 빼자!