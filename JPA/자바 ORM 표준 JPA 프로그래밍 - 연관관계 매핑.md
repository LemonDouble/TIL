# 자바 ORM 표준 JPA 프로그래밍 - 연관관계 매핑 기초

분류:  JPA
작성일시: 2021년 7월 25일 오후 2:41

## 1. 개요

- 목표
    - 객체-테이블 연관관계 차이의 이해
    - 객체의 참조와, 테이블의 FK(외래 키)를 매핑

- 용어정리
    - 방향(Direction): 단방향, 양방향
    - 다중성(Multiplicity): 다대일(N:1), 일대다(1:N), 일대일(1:1), 다대다(N:M)
    - 연관관계의 주인(Owner): 객체 양방향 연관관계는 관리 주인이 필요

- 예제 시나리오
    - 회원과 팀
    - 회원은 하나의 팀에만 소속 가능
    - 회원과 팀은 다대일 관계

![%E1%84%8C%E1%85%A1%E1%84%87%E1%85%A1%20ORM%20%E1%84%91%E1%85%AD%E1%84%8C%E1%85%AE%E1%86%AB%20JPA%20%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%80%E1%85%B3%E1%84%85%E1%85%A2%E1%84%86%E1%85%B5%E1%86%BC%20-%20%E1%84%8B%E1%85%A7%E1%86%AB%E1%84%80%E1%85%AA%E1%86%AB%E1%84%80%E1%85%AA%E1%86%AB%E1%84%80%E1%85%A8%20%E1%84%86%E1%85%A2%E1%84%91%E1%85%B5%E1%86%BC%200f6b1328dbbc483082dfcf513c9ef345/Untitled.png](https://github.com/LemonDouble/TIL/blob/main/JPA/img/Untitled%209.png)

- 즉, 객체를 테이블에 맞춰 데이터 중심으로 모델링하면, 협력 관계를 만들 수 없다.
    - 객체 지향적으로 설계하려면, Reference를 통하여 연관된 객체를 찾아야 한다.
    - 지금같은 설계의 경우, 테이블은 FK로 JOIN을 사용하여 연관된 테이블을 찾는다.

## 2. 단방향 연관관계

![%E1%84%8C%E1%85%A1%E1%84%87%E1%85%A1%20ORM%20%E1%84%91%E1%85%AD%E1%84%8C%E1%85%AE%E1%86%AB%20JPA%20%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%80%E1%85%B3%E1%84%85%E1%85%A2%E1%84%86%E1%85%B5%E1%86%BC%20-%20%E1%84%8B%E1%85%A7%E1%86%AB%E1%84%80%E1%85%AA%E1%86%AB%E1%84%80%E1%85%AA%E1%86%AB%E1%84%80%E1%85%A8%20%E1%84%86%E1%85%A2%E1%84%91%E1%85%B5%E1%86%BC%200f6b1328dbbc483082dfcf513c9ef345/Untitled%201.png](https://github.com/LemonDouble/TIL/blob/main/JPA/img/Untitled%2010.png)

## 3. 양방향 연관관계와 연관관계의 주인

- 양방향 매핑 : 서로 왔다갔다 할 수 있게! (Team → Member, Member → Team)

![%E1%84%8C%E1%85%A1%E1%84%87%E1%85%A1%20ORM%20%E1%84%91%E1%85%AD%E1%84%8C%E1%85%AE%E1%86%AB%20JPA%20%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%80%E1%85%B3%E1%84%85%E1%85%A2%E1%84%86%E1%85%B5%E1%86%BC%20-%20%E1%84%8B%E1%85%A7%E1%86%AB%E1%84%80%E1%85%AA%E1%86%AB%E1%84%80%E1%85%AA%E1%86%AB%E1%84%80%E1%85%A8%20%E1%84%86%E1%85%A2%E1%84%91%E1%85%B5%E1%86%BC%200f6b1328dbbc483082dfcf513c9ef345/Untitled%202.png](https://github.com/LemonDouble/TIL/blob/main/JPA/img/Untitled%2011.png)

- 객체와 테이블의 양방향 매핑 차이점
    - 객체의 경우,  양방향 매핑을 위해서는
        - Team에서는 Member의 ArrayList를
        - Member에서는 Team의 Member Reference가 필요하다.

    - 테이블의 경우, FK 하나로 JOIN하면 양방향 매핑이 가능하다.

- mappedBy
    - 객체와 테이블간에 연관관계를 맺는 차이를 이해해야 한다
        - 위의 경우, 객체는 연관관계가 2개이다. (단방향 2개로 양방향을 흉내낸 것이다.)
            - Member → Team 1개 (단방향)
            - Team → Member 1개 (단방향)
        - 위의 경우, 테이블은 연관관계가 1개이다.
            - Member < - > Team의 연관관계 1개 (양방향) - Member의 FK

    - 객체의 경우, 양방향 관계가 아닌 서로 다른 단방향 관계 2개이다.
        - 즉, 객체를 양방향으로 참조하려면 단방향 연관관계를 2개 만들어야 한다.
        - A → B (a.getB());
        - B → A (b.getA());

         

        ```java
        class A{
        	B b;
        }

        class B{
        	A a;
        }
        ```

    - 테이블의 경우, 외래 키 하나로 두 테이블의 연관관계를 관리한다.
        - MEMBER.TEM_ID FK 하나로 양방향으로 JOIN 할 수 있다.

        ```sql
        SELECT *
        FROM MEMBER M
        JOIN TEAM T ON M.TEAM_ID = T.TEAM_ID

        SELECT *
        FROM TEAM T
        JOIN MEMBER M ON T.TEAM_ID = M.TEAM_ID
        ```

![%E1%84%8C%E1%85%A1%E1%84%87%E1%85%A1%20ORM%20%E1%84%91%E1%85%AD%E1%84%8C%E1%85%AE%E1%86%AB%20JPA%20%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%80%E1%85%B3%E1%84%85%E1%85%A2%E1%84%86%E1%85%B5%E1%86%BC%20-%20%E1%84%8B%E1%85%A7%E1%86%AB%E1%84%80%E1%85%AA%E1%86%AB%E1%84%80%E1%85%AA%E1%86%AB%E1%84%80%E1%85%A8%20%E1%84%86%E1%85%A2%E1%84%91%E1%85%B5%E1%86%BC%200f6b1328dbbc483082dfcf513c9ef345/Untitled%203.png](https://github.com/LemonDouble/TIL/blob/main/JPA/img/Untitled%2012.png)

- 만약 Member 한명이, Team 1에서 Team 2로 이적하는 경우
    - Member 객체에서 Team을 바꿔야 될지?
    - Team 객체에서 List members 안의 내용을 바꿔야 할지?
    - 만약 그러면, Member는 Update 했는데, Team의 member는 Update 안 됐다면, FK 바꿔야 할까? 바꾸지 말아야 할까?
    - 객체와 테이블의 패러다임이 다르므로 이런 딜레마 발생한다!
    - 즉, FK 관리할 주체를 정해야 한다!

- 연관 관계의 주인(Owner)
    - 양방향에서의 매핑 규칙
    - 객체의 두 관계중 하나를 연관관계의 주인(Owner로 지정)
    - 연관관계의 주인만이 외래 키를 관리(Member 객체는 등록, 수정)
    - 주인이 아닌쪽은 읽기만 가능 (Team 객체는 읽기만 가능!)
    - 주인은 mappedBy 속성 사용 X
    - 주인이 아니라면 mappedBy 속성으로 주인 지정

- 그럼 누구를 주인으로 하는가?
    - FK 있는 곳을 주인으로 정한다!
        - 더 쉽게 이야기하면, N : 1인 곳에서 N인 곳이 무조건 FK
        - 이유 1 : 성능 이슈.
        - 이유 2 : 만약 Team을 주인으로 정하면, Team을 업데이트하는데 다른 테이블에 Update 쿼리 나간다 → 유지보수 어렵고 헷갈린다!
    - 예를 들어서, 지금의 경우는 Member가 연관관계의 주인 (Member.team)

![%E1%84%8C%E1%85%A1%E1%84%87%E1%85%A1%20ORM%20%E1%84%91%E1%85%AD%E1%84%8C%E1%85%AE%E1%86%AB%20JPA%20%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%80%E1%85%B3%E1%84%85%E1%85%A2%E1%84%86%E1%85%B5%E1%86%BC%20-%20%E1%84%8B%E1%85%A7%E1%86%AB%E1%84%80%E1%85%AA%E1%86%AB%E1%84%80%E1%85%AA%E1%86%AB%E1%84%80%E1%85%A8%20%E1%84%86%E1%85%A2%E1%84%91%E1%85%B5%E1%86%BC%200f6b1328dbbc483082dfcf513c9ef345/Untitled%204.png](https://github.com/LemonDouble/TIL/blob/main/JPA/img/Untitled%2013.png)

- 양방향 매핑시 가장 많이 하는 실수 ( 역방향 연관관계 설정 )

```java
Team team = new Team();
team.setName("TeamA");
em.persist(team);

//Member는 Owner인데, TEAM_ID 설정 안 해줬다!
Member member = new Member();
member.setName("member1");

//역방향(주인이 아닌 방향)만 연관관계 설정
//해당 내용은 실제로 반영되지 않음!
team.getMembers().add(member);

em.persist(member);

// ID    |   USERNAME    | TEAM_ID
//  1    |    member1    |   null
```

- 양방향 매핑시에는, 연관관계의 주인에 값을 입력해야 한다
(순수한 객체 관계를 고려했을 땐, 항상 양쪽 다 값을 입력하자)

```java
Team team = new Team();
team.setName("TeamA");
em.persist(team);

Member member = new Member();
member.setName("member1");

//연관관계의 주인에 값 설정
member.setTeam(team); 
//객체 관계를 고려해, Team에도 값 설정
team.getMembers().add(member);
em.persist(member);
```

- 양방향 연관관계 설정 - 주의점
    - 순수 객체 상태를 고려해 항상 양쪽에 값을 설정하자!
        - Owner에만 값을 설정해 줬을 때의 문제되는 경우

        ```java
        //이때, 영속성 컨텍스트에 1차 Cache 된다.
        // TEAM_ID | TEAM_NAME | MEMBERS
        //    1    |  TeamA    |   null
        Team team = new Team();
        team.setName("TeamA");
        em.persist(team);

        Member member = new Member();
        member.setName("member1");

        //연관관계의 주인에 값 설정
        member.setTeam(team);

        //객체 관계를 고려하지 않고 Team에는 값을 주지 않았다
        //team.getMembers().add(member);

        //만약 여기서 em.flush(), em.clear(), 후에 사용하면 괜찮지만
        //그렇지 않은 경우에는?

        //이떄, 1차 Cache된 값을 그대로 가져온다
        // TEAM_ID | TEAM_NAME | MEMBERS
        //    1    |  TeamA    |   null
        Team findTEam = em.find(Team.class, team.getId());

        //아직 Update 반영되지 않았으므로 Null! (Error)
        List<Member> members = findTeam.getMembers
        ```

        - Test Case 작성할때 → JPA 없이 순수한 JAVA 코드로 동작하는 경우, 양방향 연관관계 세팅이 안 되어있다면 테스트하기 어렵다.
    - 연관관계 편의 메서드를 생성하자!
        - 다음과 같이 코드 두 개를 항상 붙여줘야 하기 때문에, Human Error가 생길 수 있다.

        ```java
        //양방향 매핑
        member.setTeam(team);
        team.getMembers().add(member);
        ```

        - 따라서, member.setTeam에 다음과 같이 세팅해두면, 항상 setTeam을 불렀을 때 양방향 연결이 보장된다.

        ```java
        public void setTeam(Team team){
        	this.team = team;

        	team.getMembers().add(this);
        }
        ```

        - 하지만 getter, setter 관례가 있으므로, 메서드 이름을 바꿔주면 좋다.

        ```java
        //이름 변경!
        public void changeTeam(Team team){
        	this.team = team;

        	team.getMembers().add(this);
        }
        ```

    - 양방향 매핑시 무한 루프를 조심하자!
        - 예 : toString(), lombok, JSON 생성 라이브러리

        ```java
        members.toString(){
        //이 때, Team 내부에서도 모든 필드에 대해 toString 호출한다!
        System.out.println("id = " + id + "name = " + name + "team = " + team)
        }

        team.toString(){
        //이때, 다시 Member를 Call하므로 무한 Loop
        System.out.println("id = " + id + "name = " + name + "member =" + member) 
        }
        ```

        - Lombok에서는 toString() 사용하지 말자
        - Spring Controller에서는 Entity 자체를 반환하지 말자
            - 무한 Loop 생길 수 있다.
            - Entity는 변경될 수 있다 → 엔티티 변경 시 API Spec이 바뀔 수 있다.
            - 엔티티 자체를 반환하지 말고, DTO 통해 반환하자

- 양방향 매핑 정리
    - 단방향 매핑만으로 먼저 연관관계 매핑을 완료해야 한다!!!!
    - 양방향 매핑은, 반대 방향으로 조회(객체 그래프 탐색) 기능이 추가된 것 뿐이다.
    - JPQL에서 역방향으로 탐색할 일이 많다.
    - 단방향 매핑을 먼저 잘 하고, 양방향은 필요할 때 추가해도 됨
    (테이블에 영향을 주지 않음)
    - 즉, 단방향으로 설계 후, 어플리케이션 개발 단계에 들어가 "정말 양방향이 필요할 때" 어쩔 수 없이 추가하는게 좋다.