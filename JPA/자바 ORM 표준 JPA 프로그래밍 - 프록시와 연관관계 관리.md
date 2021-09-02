# 자바 ORM 표준 JPA 프로그래밍 - 프록시와 연관관계 관리

분류:  JPA
작성일시: 2021년 7월 27일 오후 3:36

## 1. 프록시

- 만약 Member 객체에 Team에 대한 Reference 있다면, Team의 정보를 항상 같이 조회해야 할까?
    - 만약, Team이 Member와 거의 항상 같이 호출된다면, 같이 조회해두는 편이 좋을 것이다
    - 만약 Team이 아주 가끔 조회되는 정도에 그친다면,

- 프록시 기초
    - em.find() : 데이터베이스를 통해 실제 엔티티 객체 조회
    - em.getReference() : 데이터베이스 조회를 미루는 가짜(프록시) 엔티티 객체 조회

- 프록시 특징
    - 실제 클래스를 상속받아 만들어짐
    - 실제 클래스와 겉 모양이 같다.
    - 사용하는 입장에서는 진짜 객체인지, 프록시 객체인지 구분하지 않고 사용하면 됨 (이론상)
    - 대신, 프록시 객체는 실제 객체의 참조(target)을 보관
    - 프록시 객체를 호출하는 경우, 객체는 실제 객체의 메소드 호출

![%E1%84%8C%E1%85%A1%E1%84%87%E1%85%A1%20ORM%20%E1%84%91%E1%85%AD%E1%84%8C%E1%85%AE%E1%86%AB%20JPA%20%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%80%E1%85%B3%E1%84%85%E1%85%A2%E1%84%86%E1%85%B5%E1%86%BC%20-%20%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%86%A8%E1%84%89%E1%85%B5%E1%84%8B%E1%85%AA%20%E1%84%8B%E1%85%A7%E1%86%AB%E1%84%80%E1%85%AA%E1%86%AB%E1%84%80%20954e9c0c2abf480e8389947239481cc1/Untitled.png](https://github.com/LemonDouble/TIL/blob/main/JPA/img/Untitled%2023.png)

- 실제 메소드가 호출될때, 영속성 컨텍스트 통해서 초기화 값을 요청

- 프록시 객체는 처음 사용할 때 한 번만 초기화
- 프록시 객체를 초기화 할 때, 프록시 객체가 실제 엔티티로 바뀌는 것은 아니다. 초기화되면 프록시 객체를 통해 실제 엔티티에 접근 가능 ( MemberProxy.getName() → Member.getName() )
- 프록시 객체는 원본 엔티티를 상속받음. 따라서 타입 체크시 유의 (== 실패, insatance of 사용)
- 영속성 컨텍스트에 찾는 엔티티가 이미 있으면, em.getReference()를 호출해도 실제 엔티티 반환
- 반대로, 처음에 em.getReference() 가 먼저 호출되었다면, em.find()로 호출해도 Proxy 반환
- 즉, 같은 객체임을 보장해 준다!

- 영속성 컨텍스트의 도움을 받을 수 없는 준영속 상태일 때, 프록시를 초기화하면 문제 발생 (하이버네이트의 경우, org.hibernate.LazyInitializationException 예외 발생)
- 즉, 초기화 전에 em.detatch()나, em.clear() 발생한 경우 문제가 발생한다!

- 프록시 확인
    - 프록시 인스턴스의 초기화 여부 확인
        - EntityManagerFactory.PersistenceUniUtil.isLoaded(Object entity)
    - 프록시 클래스 확인 방법
        - entity.getClass().getName()
    - 프록시 강제 초기화
        - org.hibernate.Hibernate.initalize(entity);

- 참고 : JPA 표준은 강제 초기화 없음
    - 강제 호출로 강제 초기화 가능 : member.getName();

## 2. 즉시 로딩과 지연 로딩

- Member에 Team의 Reference 있지만, 비즈니스 로직상 Member만 조회하고 Team은 조회하지 않는 경우

- 지연 로딩(Lazy Loading)을 사용해 프록시로 조회

```java
```

- Team의 엔티티는 Proxy를 이용하여 조회된다!
- 이렇게 함으로써, Member를 로딩할 때 Team은 로드되지 않는다!
- Team은, Team 내부에 있는 데이터를 조회하는 경우에만 쿼리를 통해 호출된다.

---

- Member와 Team이 비즈니스 로직상 거의 항상 호출되는 경우

- 즉시 로딩 EAGER 을 사용하여 함께 조회!

```java
```

- Hybernate는 JOIN을 사용해 SQL을 한번에 조회한다.

- 프록시와 즉시 로딩 주의
    - 즉시 로딩은 사용하지 말 것 !!
    - 가급적 모두 지연 로딩으로 통일(특히 실무에서)
    - 즉시 로딩을 적용하는 경우, 예상하지 못한 SQL이 발생
        - 예를 들어 Member만 em.find 했는데, Team에 대한 Join이 추가로 발생
    - 즉시 로딩은 JPQL에서 N+1 문제를 일으킨다.
        - 처음 Query가 1개일때, 해당 쿼리로 인해 새 쿼리가 N개가 더 나가면 N+1 문제라고 한다.
        - 예를 들어 select * from Member을 JPQL로 보냈을 경우, EAGER로 설정되어 있다면 모든 Member의 개수만큼 Query 나간다!
    - @ManyToOne, @OneToOne은 기본이 즉시 로딩(EAGER) → LAZY로 설정
        - ~~ToOne은 기본이 즉시 로딩
    - @OneToMany, @ManyToMany는 기본이 지연 로딩

    - 즉시 로딩이 필요한 경우, JPQL fetch 조인이나, 엔티티 그래프 기능을 사용해라!

## 3. 영속성 전이 : CASCADE

- 영속성 전이: CASCADE
    - 특정 엔티티를 영속 상태로 만들고 싶을 때, 연관된 엔티티도 같이 영속 상태로 만들고 싶을 때
    - 예: 부모 엔티티를 저장할 때 자식 엔티티도 함께 저장

```java
```

```java
```

- CASCADE 주의점
    - 영속성 전의는 연관관계 매핑과 아무 관련이 없다.
    - 엔티티를 영속화할 때, 연관된 엔티티도 함께 영속화하는 편리함을 제공할 뿐이다!

- CASCADE의 종류
    - ALL : 모두 적용
    - PERSIST : 영속
    - REMOVE : 삭제
    - MERGE : 병합
    - REFRESH : REFRESH
    - DETACH : DETACH

- CASCADE를 언제 사용하는가?
    - 하나의 부모가 자식들을 전부 관리할때 (소유자가 하나일 때)
    - 예시 : 게시판, 첨부파일 테이블 데이터 등

    - 만약 파일을 여러 엔티티가 관리한다면? 사용하면 안 된다!

## 4. 고아 객체

- 고아 객체 : 부모와의 연관관계가 끊어진 자식 엔티티

- orphanRemoval = true (고아 객체 자동 제거)
    - 참조게 제거된 엔티티는 다른 곳에서 참고하지 않는 고아 객체로 보고 삭제하는 기능
    - 참조하는 곳이 하나일 때 사용해야 함!
    - 특정 엔티티가 개인 소유할 때 사용
    - @OneToOne, @OneToMany만 가능

```java
@OneToMnay(mappedBy="parent", orphanRemoval = true)

parent1.getChildren().remove(0); //자식 클래스를 컬렉션에서 제거
//자동으로 자식 엔티티에 DELETE QUERY 나간다!
```

- 참고 : 개념적으로 부모를 제거하면 자식은 고아가 된다. 따라서, 고아 객체 제거 기능을 활성하는 경우, 부모가 제거되면 자식도 제거된다. 이것은 CascadeType.REMOVE처럼 동작한다.

## 5. 영속성 전이 + 고아 객체, 생명 주기

- CascadeType.ALL + orphanRemoval=true
    - 스스로 생명주기를 관리하는 엔티티는 em.persist()로 영속화, em.remove()로 제거
    - 두 옵션을 모두 활성화하면, 부모 엔티티를 통해 자식의 생명주기를 관리할 수 있음.
    - 도메인 주도 설계(DDD)의 Aggregate Root 개념을 구현할 때 유용
