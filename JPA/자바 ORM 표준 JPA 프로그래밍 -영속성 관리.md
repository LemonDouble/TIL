# 자바 ORM 표준 JPA 프로그래밍 -영속성 관리

분류:  JPA
작성일시: 2021년 7월 24일 오후 1:56

## 1. 영속성 컨텍스트

- 영속성 컨텍스트와 트랜잭션의 주기를 맞춰 주자

- JPA를 사용시
    - EntityManagerFatory → 웹 어플리케이션 하나마다 생성
    - EntityManager → 고객의 요청이 오면 생성
        - Connection pool 사용하여 DB에 연결함

- 영속성 컨텍스트
    - JPA를 이해하는데 가장 중요한 용어
    - "엔티티를 영구 저장하는 환경"
    - EntityManager.persist(entity);

    - 영속성 컨텍스트는 논리적인 개념, 눈에 보이지 않는다.
    - 엔티티 매니저를 통해 영속성 컨텍스트에 접근

![%E1%84%8C%E1%85%A1%E1%84%87%E1%85%A1%20ORM%20%E1%84%91%E1%85%AD%E1%84%8C%E1%85%AE%E1%86%AB%20JPA%20%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%80%E1%85%B3%E1%84%85%E1%85%A2%E1%84%86%E1%85%B5%E1%86%BC%20-%E1%84%8B%E1%85%A7%E1%86%BC%E1%84%89%E1%85%A9%E1%86%A8%E1%84%89%E1%85%A5%E1%86%BC%20%E1%84%80%E1%85%AA%E1%86%AB%E1%84%85%E1%85%B5%205e9783b0ebd242b18247d36a13680ef0/Untitled.png](https://github.com/LemonDouble/TIL/blob/main/JPA/img/Untitled%205.png)

- 엔티티의 생명 주기
    - 비영속 (new/transient)
        - 영속성 컨텍스트와 전혀 관계가 없는 새로운 상태

        ```java
        //객체를 생성만 한 상태
        Member member = new Member();
        member.setId("member1");
        member.setUsername("user1");
        ```

    - 영속 (managed)
        - 영속성 컨텍스트에 의해 관리되는 상태

        ```java
        EntityManager em = emf.createEntityManager();
        em.getTransaction().begin();

        //객체를 저장한 상태 (영속)
        em.persist(member);

        //아직 DB에 저장되진 않는다. 
        ```

    - 준영속 (detached)
        - 영속성 컨텍스트에 저장되었다가 분리된 상태
    - 삭제(removed)
        - 삭제된 상태

![%E1%84%8C%E1%85%A1%E1%84%87%E1%85%A1%20ORM%20%E1%84%91%E1%85%AD%E1%84%8C%E1%85%AE%E1%86%AB%20JPA%20%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%80%E1%85%B3%E1%84%85%E1%85%A2%E1%84%86%E1%85%B5%E1%86%BC%20-%E1%84%8B%E1%85%A7%E1%86%BC%E1%84%89%E1%85%A9%E1%86%A8%E1%84%89%E1%85%A5%E1%86%BC%20%E1%84%80%E1%85%AA%E1%86%AB%E1%84%85%E1%85%B5%205e9783b0ebd242b18247d36a13680ef0/Untitled%201.png](https://github.com/LemonDouble/TIL/blob/main/JPA/img/Untitled%206.png)

- 영속성 컨텍스트의 이점
    - 1차 캐시
        - key : PK로 입력한 값
        - value : 해당 object
        - 예를 들어, key : "1L", Value(Entity) : member

        - 만약 persist 후 바로 find 한다면, DB 연결하지 않고 1차 Cache에서 바로 조회
        - 만약 1차 캐시에 없는 값 (예, member2)  find한다면, DB에서 조회하여 1차 캐시에 저장 후 해당 object를 반환
        - 이후, 동일한 값 (member2) find 한다면 1차 캐시에서 찾아서 반환한다.

        - 하지만 성능에 크게 도움이 되진 않는다!
            - EntityManager는 Transaction 단위로 만든다.
            - Transaction이 종료되면 EntityManager도 종료 (1차 캐시도 사라진다)
            - 한 Transaction 안 (매우 짧은 순간 내)에서만 사용된다!
            - 비즈니스가 엄청 복잡할 땐 도움이 될 수도?? 있다

    - 동일성(identity) 보장
        - 1차 캐시로 반복 가능한 읽기(REPEATABLE READ) 등급의 트랜잭션 격리 수준을 데이터베이스가 아닌 어플리케이션 차원에서 제공

        ```java
        Member a = em.find(Member.class, "member1");
        Member b = em.find(Member.class, "member1");

        System.out.println(a == b); //동일성 비교 true
        ```

    - 트랜잭션을 지원하는 쓰기 지연 (transactinal write-behind)
        - Buffering 가능
        - 이렇게 트랜잭션을 모아 보내면 최적화의 여지가 생긴다!

    ```java
    EntityTransaction tx = em.getTransaction();
    //엔티티 매니저는 변경시 트랜잭션을 시작해야 한다.

    tx.begin(); //트랜잭션 시작

    em.persist(memberA);
    em.persist(memberB);
    //여기까지 INSERT SQL을 데이터베이스에 보내지 않는다.

    //커밋하는 순간 데이터베이스에 INSERT SQL을 보낸다
    tx.commit(); //트랜잭션 커밋
    ```

    ![%E1%84%8C%E1%85%A1%E1%84%87%E1%85%A1%20ORM%20%E1%84%91%E1%85%AD%E1%84%8C%E1%85%AE%E1%86%AB%20JPA%20%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%80%E1%85%B3%E1%84%85%E1%85%A2%E1%84%86%E1%85%B5%E1%86%BC%20-%E1%84%8B%E1%85%A7%E1%86%BC%E1%84%89%E1%85%A9%E1%86%A8%E1%84%89%E1%85%A5%E1%86%BC%20%E1%84%80%E1%85%AA%E1%86%AB%E1%84%85%E1%85%B5%205e9783b0ebd242b18247d36a13680ef0/Untitled%202.png](https://github.com/LemonDouble/TIL/blob/main/JPA/img/Untitled%207.png)

    - 변경 감지 (Dirty Checking)

        ```java
        EntityTransaction tx = em.getTransaction();
        tx.begin(); //트랜잭션 시작

        //영속 엔티티 조회
        Member memberA = em.find(Member.class, "memberA");

        //영속 엔티티 데이터 수정
        memberA.setUsername("hi");
        memberA.setAge(10);

        //em.update(member) 이런 코드가 없어도 작동한다!

        tx.commit(); //트랜잭션 커밋
        ```

    ![%E1%84%8C%E1%85%A1%E1%84%87%E1%85%A1%20ORM%20%E1%84%91%E1%85%AD%E1%84%8C%E1%85%AE%E1%86%AB%20JPA%20%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%80%E1%85%B3%E1%84%85%E1%85%A2%E1%84%86%E1%85%B5%E1%86%BC%20-%E1%84%8B%E1%85%A7%E1%86%BC%E1%84%89%E1%85%A9%E1%86%A8%E1%84%89%E1%85%A5%E1%86%BC%20%E1%84%80%E1%85%AA%E1%86%AB%E1%84%85%E1%85%B5%205e9783b0ebd242b18247d36a13680ef0/Untitled%203.png](https://github.com/LemonDouble/TIL/blob/main/JPA/img/Untitled%208.png)

    - 1차 캐시 : id, Entity, Snapshot 세가지 Field가 있다.
        - Snapshot : 최초 데이터가 들어온 최초의 상태 (DB에서 Read되거나.. em.persist로 저장되거나..)
    - 지연 로딩 (Lazy Loading)

## 2. 플러시

- 영속성 컨텍스트의 변경 내용을 데이터베이스에 반영

- 플러시가 발생되는 경우 생기는 일
    - 변경 감지가 일어난다.
    - 수정된 엔티티를 쓰기 지연 SQL 저장소에 등록한다.
    - 쓰기 지연 SQL 저장소의 쿼리를 DB에 전송한다. (등록, 수정, 삭제 쿼리)

- 영속성 컨텍스트를 플러시하는 방법
    - em.flush() : 직접 호출
    - 트랜잭션 커밋 : 플러시 자동 호출
    - JPQL 쿼리 실행 : 플러시 자동 호출
        - JPQL은 DB에 Request를 날리는데, JPQL 쿼리 실행 전에 Flush가 되지 않는다면 데이터 무결성에 문제가 생긴다!

- Flush는!
    - 영속성 컨텍스트를 비우지 않음
    - 영속성 컨텍스트의 변경내용을 DB와 동기화
    - Transaction이라는 작업 단위가 중요, 커밋 직전에만 동기화하면 됨

## 3. 준영속 상태

- 영속 → 준영속
- 영속 상태의 엔티티가 영속성 컨텍스트에서 분리 (detached)
- 영속성 컨텍스트가 제공하는 기능을 사용하지 못함

- 준영속 상태로 만드는 방법
    - em.detach(entity) : 특정 엔티티만 준영속 상태로 전환
    - em.clear() : 영속성 컨텍스트를 완전히 초기화
    - em.close() : 영속성 컨텍스트를 종료

```java
EntityTransaction tx = em.getTransaction();
tx.begin();

//영속 엔티티 조회
Member memberA = em.find(Member.class, "memberA");

//영속 엔티티 데이터 수정
memberA.setUsername("hi");

//준영속 상태로 수정
em.detatch(memberA);

tx.commit(); //트랜잭션 커밋

//memberA는 수정되지 않는다!
```
