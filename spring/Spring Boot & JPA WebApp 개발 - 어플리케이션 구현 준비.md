# Spring Boot & JPA WebApp 개발 - 어플리케이션 구현 준비

분류: SPRING+JPA
작성일시: 2021년 8월 20일 오후 9:40

## 1. 구현 요구사항

- 회원 기능

  - 회원 등록
  - 회원 조회

- 상품 기능

  - 상품 등록
  - 상품 수정
  - 상품 조회

- 주문 기능

  - 상품 주문
  - 주문 내역 조회
  - 주문 취소

- 예제를 단순화하기 위해 다음 기능은 구현하지 않는다.
  - 로그인과 권한 관리
  - 파라미터 검증과 예외처리
  - 상품은 도서만 사용
  - 카테고리는 사용하지 않음
  - 배송 정보는 사용하지 않음.

## 2. 어플리케이션 아키텍쳐

![Untitled](https://github.com/LemonDouble/TIL/blob/main/spring/img/Untitled%2026.png)

- 계층형 구조 사용

  - Controller, web : 웹 계층
  - service : 비즈니스 로직, 트랜잭션 처리
  - repository: JPA를 직접 사용하는 계층, 엔티티 매니저 사용
  - domain : 엔티티가 모여있는 계층, 모든 계층에서 사용

- 패키지 구조

  - jpabook.jpashop
    - domain
    - exception
    - repository
    - service
    - web

- 개발 순서 : 서비스, 리포지토리 계층 개발 → Test Case로 검증 → 웹 계층 적용
