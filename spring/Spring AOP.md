# Spring: AOP

분류: Spring
작성일시: 2021년 7월 6일 오후 8:17

## 1. AOP가 필요한 상황

- 예시 : 모든 메소드의 호출 시간을 측정하고 싶다면?
- Brute한 방법 : 모든 함수에 시간 재는 코드를 추가

    → 문제 : 시간 측정 로직은 핵심 관심 사항이 아니지만, 핵심 비즈니스 코드와 섞여 관리가 힘들다

## 2. AOP 적용

- AOP : Aspect Oriencted Programming
- 공통 관심 사항(cross-cutting concern)과 핵심 관리 사항(core concern) 분리

 1. aop/TimeTraceAop 클래스 추가 (aop 폴더 생성)

```java
```

@Around "execution" 부분 : 실행될 패키지, 파라미터 등을 관리

## 3. Spring에서 AOP의 동작 방식

![Spring%20AOP%203d2afff0542a4cdf8bcd81a8c1cdb1fa/Untitled.png](https://github.com/LemonDouble/TIL/blob/main/spring/img/Untitled%204.png)

- 가짜 Spring Bean (Proxy)를 먼저 호출하고, Proxy가 끝나면 실제 Cotnroller를 호출

```java
```

- getClass 메소드를 이용하여 위와 같이, 실제로 프록시가 적용되는지 확인해 볼 수 있다!
