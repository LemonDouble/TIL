# Spring Boot & JPA WebApp 개발 - 주문 도메인 개발

분류: SPRING+JPA
작성일시: 2021년 8월 23일 오후 3:18

## 1. 개요

- 구현 기능
  - 상품 주문
  - 주문내역 조회
  - 주문 취소

## 2. 주문, 주문상품 엔티티 개발

- 왜 생성자 말고 static으로 선언한 생성 메소드를 쓰나요?

  - [https://johngrib.github.io/wiki/static-factory-method-pattern/](https://johngrib.github.io/wiki/static-factory-method-pattern/)
  - 정적 팩토리 메서드(static factory method) 참조

- 비즈니스 로직과 생성 메소드를 추가한다.

- jpashop.domain.item.order.class 수정

```java
```

- jpashop.domain.OrderItem.class 수정

```java
```

## 3. 주문 리포지토리 개발

- jpashop.repository.OrderRepository.class

```java
```

## 4. 주문 서비스 개발

- jpashop.service.OrderService.class

```java
```

- 지금의 경우, 도메인에 핵심 비즈니스 로직이 있고, 서비스 계층은 단순히 엔티티에 필요한 요청을 위임하는 역할만 하고 있다.
- 이처럼 엔티티가 비즈니스 로직을 가지고, 객체 지향의 특성을 확인하는 방법을 도메인 모델 패턴 ( [https://martinfowler.com/eaaCatalog/domainModel.html](https://martinfowler.com/eaaCatalog/domainModel.html) ) 라고 한다.

- 반대로, 엔티티에는 비즈니스 로직이 거의 없고, 서비스 계층에서 대부분의 비즈니스 로직을 처리하는 것을 트랜잭션 스크립트 패턴이라고 한다. ([https://martinfowler.com/eaaCatalog/transactionScript.html](https://martinfowler.com/eaaCatalog/transactionScript.html))
- Domain은 getter, setter밖에 없고, 서비스 단에서 직접 SQL 쿼리를 날리는 경우를 생각해 보자!

## 5. 주문 기능 테스트

- test/.../jpashop.service.OrderServiceTest.class

```java
```

## 6. 주문 검색 기능 개발

- 회원 이름, 주문상태 가지고 찾는거 만듬!

![Untitled](https://github.com/LemonDouble/TIL/blob/main/spring/img/Untitled%2026.png)

- jpashop.repository.OrderSearch.class

```java
```

- OrderRepository.class 의 findAll() 함수 수정

```java
```

- OrderService.class 의 findOrders() 함수 수정

```java
```

- OrderServiceTest.class에서 새로 Test 해서 검증

```java
```
