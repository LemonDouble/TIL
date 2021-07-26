# 자바 ORM 표준 JPA 프로그래밍 - 다양한 연관관계 매핑

분류:  JPA
작성일시: 2021년 7월 25일 오후 6:42

## 1. 연관관계 매핑 시 고려사항 3가지

1. 다중성

    (데이터베이스 기준)

    - 다대일: @ManyToOne
    - 일대다: @OneToMany
    - 일대일: @OneToOne
    - 다대다: @ManyToMany - 실무에서는 사용하면 안 된다!

    - 만약 관계가 어렵다면, 관계를 뒤집어서 생각해 보자
        - Member ?? Team이라면, Team → Member는 일대다 관계이다.
        - 따라서 Member → Team은 다대일 관계이다 ( 대칭성 )

1. 단방향, 양방향
    - 테이블
        - FK 하나로 양쪽 조인 가능
        - 사실, 방향이라는 개념 자체가 없음 (기본이 양방향)
    - 객체
        - 참조용 필드가 있는 쪽으로만 참조 가능
        - 한쪽만 참조하면 단방향
        - 양쪽이 서로 참조하면 양방향 (실제로는 단방향밖에 없지만, 이해를 위해 개념상)

1. 연관관계의 주인
    - 테이블은 FK 하나로 두 테이블이 연관관계를 맺음
    - 객체 양방향 관계는 A→B, B→A처럼 참조가 2군데
    - 객체 양방향 관계는 참조가 2군데이므로, 둘 중 테이브의 FK를 관리할 곳을 명확히 지정해줘야 함.
    - 연관관계의 주인 : 외래 키를 관리하는 참조(Reference)
    - 주인의 반대편 : 외래 키에는 영향을 주지 않음. 단순 조회만 가능

## 2. 다대일 (N:1)

- Member (N) 과 Team (1)의 관계
- 가장 많이 사용하는 연관관계, 다대일의 반대는 일대다

- 1 쪽에 FK가 들어가야 한다 (Member에 TEAM_ID가 FK로 들어가야 한다.)

- Member(N)

```java
public class Member{
	...
	//Member -> Team!
	@ManyToOne
	@JoinColumn(name = "TEAM_ID")
	private Team team;
	...
}
```

- Team(1)

```java
public class Team{
	@Id @GeneratedValue
	@Column(name = "TEAM_ID")
	private Long id;
	...

	//양방향 추가, DB는 변경되는 부분이 없다!
	//이 리스트는 ReadOnly
	@OneToMany(mappedBy="team")
	private List<Member> members = new ArrayList<>();
	
}
```

## 3. 일대다 (1:N)

- 권장되지는 않음!

![%E1%84%8C%E1%85%A1%E1%84%87%E1%85%A1%20ORM%20%E1%84%91%E1%85%AD%E1%84%8C%E1%85%AE%E1%86%AB%20JPA%20%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%80%E1%85%B3%E1%84%85%E1%85%A2%E1%84%86%E1%85%B5%E1%86%BC%20-%20%E1%84%83%E1%85%A1%E1%84%8B%E1%85%A3%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20%E1%84%8B%E1%85%A7%E1%86%AB%E1%84%80%E1%85%AA%E1%86%AB%E1%84%80%E1%85%AA%20229b4ad62486498ea1124401db18e7bc/Untitled.png](https://github.com/LemonDouble/TIL/blob/main/JPA/img/Untitled%2016.png)

- 일대다 단방향 정리
    - 일대다 단방형은, 일대다(1:N)에서 일(1)이 연관관계의 주인
    - 테이블 일대다 관계는, 항상 다(N) 쪽에 외래 키가 있음.
    - 객체와 테이블의 차이 때문에, 반대편 테이블의 외래 키를 관리하는 특이한 구조
    - @JoinColumn을 꼭 사용해야 함. 그렇지 않으면 조인 테이블 방식을 사용 (중간에 테이블을 하나 추가함)
        - 아니면 TEAM_ID, MEMBER_ID 관리하는 중간 Table 생성되는 방식으로 동작
    - 단점
        - 엔티티가 관리하는 외래 키가 다른 테이블에 있음
        - 연관관계 관리를 위해 추가로 UPDATE SQL 실행

    - 가능한 일대다보단 다대일 양방향 매핑을 사용하는것이 좋다.

- Member(N)

```java
public class Member{
	@Id @GeneratedValue
	@Column(name="MEMBER_ID")
	private Long id;

	@Column(name="USERNAME")
	private String username;

	//Getter , Setter
```

- Team (1)

```java
public class Team{
	@Id @GeneratedValue
	@Column(name = "TEAM_ID")
	private Long id;
	...
	
	//@JoinColumn : 외래 키를 매핑할 때 사용
	//name = 매칭할 외래 키 이름을 지정
	@OneToMnay
	@JoinColumn(name = "TEAM_ID")
	private List<Member> members = new ArrayList<>();
	
}
```

- Team에 Member를 추가하면, 해당 Member를 찾아서 해당 Member에 Update Query 나간다.
- 하지만 Code에서, Team만 손을 댔는데 Member 테이블이 업데이트되니 혼란스러울 수 있다.
- 다대일 양방향 매핑시 비슷한 기능 구현할 수 있다..

## 4. 일대일 (1:1)

- 일대일 관계는 그 반대도 일대일

- 주 테이블이나, 대상 테이블 중에 외래 키 선택 가능
    - 주 테이블(Member)에 외래 키
    - 대상 테이블(Team)에 외래 키
- 외래 키에 데이터베이스 유니크(UNI) 제약 조건 추가

- 예를 들어, Member는 단 하나의 Locker(사물함) 가질 수 있다고 가정

![%E1%84%8C%E1%85%A1%E1%84%87%E1%85%A1%20ORM%20%E1%84%91%E1%85%AD%E1%84%8C%E1%85%AE%E1%86%AB%20JPA%20%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%80%E1%85%B3%E1%84%85%E1%85%A2%E1%84%86%E1%85%B5%E1%86%BC%20-%20%E1%84%83%E1%85%A1%E1%84%8B%E1%85%A3%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20%E1%84%8B%E1%85%A7%E1%86%AB%E1%84%80%E1%85%AA%E1%86%AB%E1%84%80%E1%85%AA%20229b4ad62486498ea1124401db18e7bc/Untitled%201.png](https://github.com/LemonDouble/TIL/blob/main/JPA/img/Untitled%2017.png)

- 단방향의 경우
    - Member가 Locker의 FK 가지고, Unique 제약 조건만 추가되면 된다.
    (다대일 단방향과 거의 같다.)
- 양방향의 경우
    - Member가 Locker의 FK 가지고,
    - Locker가 Member의 Reference 가진다.

- 다대일처럼, 외래키가 있는 곳이 연관관계의 주인
- 반대편의 경우에는, mappedBy 적용

- Member.class

```java
public class Member{
	...

	@OneToOne
	@JoinColumn(name = "LOCKER_ID")
	private Locker locker;

	//Getter , Setter
```

- Locker.class

```java
@Entity
public class Locker{
	@Id @GeneratedValue
	private Long id;

	private String name;

	//양방향 만들 시
	@OneToOne(mappedBy = "locker")
	private Member member;

}
```

- FK를 누가 들고 있을 것인가?
    - Member가 LOCKER_ID를 들고 있을 경우
        - 만약 한 Locker를 여러 Member가 공유할 수 있다면?
        - Member에서 Unique 제약 조건 빼면 된다!

        - 성능상으로 유리 
        (Member가 Locker보다 Select 자주 되므로, 두번 Select 할 필요가 없다)
        - 대신 Member에 여러 정보 들어가게 되므로, Member 데이터 자체가 커질 수도 있다.
    - Locker가 MEMBER_ID 들고 있는 경우
        - 만약 한 사람이 여러 Locker 가질 수 있도록 바뀐다면?
        - Locker에서 Unique 제약 조건만 빼면 된다!

- 정리
    - 주 테이블(자주 Access하는 객체)에 외래 키를 가지는 경우
        - 주 객체가 대상 객체의 참조를 가지는 것 처럼, 주 테이블에 외래 키를 두고 대상 테이블을 찾음
        - 객체지향 개발자 선호
        - JPA 매핑 편리
        - 장점 : 주 테이블만 조회해도, 대상 테이블에 데이터가 있는지 확인 가능
        (Locker 있으면 값 있음, 없으면 null)
        - 단점 : 값이 없으면 FK에 null을 허용한다.

    - 대상 테이블에 외래 키
        - 대상 테이블에 외래 키가 존재 (Locker)
        - 전통전인 DB 개발자 선호
        - 장점 : 주 테이블과 대상 테이블을 일대일에서 일대다 관계로 변경할 때 테이블 구조 유지
        - 단점 : 프록시 기능의 한계로 지연 로딩으로 설정해도 항상 즉시 로딩됨

## 5. 다대다 (N:M)

- 실제 케이스에선 사용하면 안 된다!!!!!

- RDB에서는 정규화된 테이블 2개로 다대다 관계를 표현할 수 없음.
- 연결 테이블을 추가해 N:M을 N:1, 1:M으로 풀어내야 함.

![%E1%84%8C%E1%85%A1%E1%84%87%E1%85%A1%20ORM%20%E1%84%91%E1%85%AD%E1%84%8C%E1%85%AE%E1%86%AB%20JPA%20%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%80%E1%85%B3%E1%84%85%E1%85%A2%E1%84%86%E1%85%B5%E1%86%BC%20-%20%E1%84%83%E1%85%A1%E1%84%8B%E1%85%A3%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20%E1%84%8B%E1%85%A7%E1%86%AB%E1%84%80%E1%85%AA%E1%86%AB%E1%84%80%E1%85%AA%20229b4ad62486498ea1124401db18e7bc/Untitled%202.png](https://github.com/LemonDouble/TIL/blob/main/JPA/img/Untitled%2018.png)

- 객체는 다대다 관계가 가능하다. (Collections)
    - Member는 Product List를 가짐
    - Product는 Member List를 가짐

- 다대다
    - @ManyToMany
    - @JoinTable로 연결 테이블 지정
    - 다대다 매핑 : 단방향, 양방향 가능

- Member.class

```java
public class Member{
	...

	//MEMBER_PRODUCT라는 중간 연결 테이블을 통해 풀어낸다
	@ManyToMany
	@JoinTable(name = "MEMBER_PRODUCT")
	private List<Product> products = new ArrayList<>();

	//Getter , Setter
```

- Product.class

```java
@Entity
public class Product{
	@Id @GeneratedValue
	private Long id;
	private String name;

	@ManyToMany(mappedBy = "products")
	private List<Member> members = new ArrayList()<>;
}
```

- 다대다 매핑의 한계
    - 연결 테이블이 단순히 연결만 하고 끝나지 않음
    - 중간 연결 테이블에 주문시간, 수량 같은 데이터가 들어올 수 있음
    - 다대다 매핑은 중간 테이블에 추가 정보를 더 넣을 수 없음!

- 다대다 한계 극복
    - 연결 테이블용 엔티티 추가(연결 테이블을 엔티티로 승격)
    - @ManyToMany → @OneToMany , @ManyToOne

![%E1%84%8C%E1%85%A1%E1%84%87%E1%85%A1%20ORM%20%E1%84%91%E1%85%AD%E1%84%8C%E1%85%AE%E1%86%AB%20JPA%20%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%80%E1%85%B3%E1%84%85%E1%85%A2%E1%84%86%E1%85%B5%E1%86%BC%20-%20%E1%84%83%E1%85%A1%E1%84%8B%E1%85%A3%E1%86%BC%E1%84%92%E1%85%A1%E1%86%AB%20%E1%84%8B%E1%85%A7%E1%86%AB%E1%84%80%E1%85%AA%E1%86%AB%E1%84%80%E1%85%AA%20229b4ad62486498ea1124401db18e7bc/Untitled%203.png](https://github.com/LemonDouble/TIL/blob/main/JPA/img/Untitled%2019.png)

- 중간 MemberProduct.class

```java
@Entity
public class MemberProduct{
	@Id, @GeneratedValue
	private Long id;

	@ManyToOne
	@JoinColumn(name="MEMBER_ID")
	private Member member;

	@ManyToOne
	@JoinColumn(name="PRODUCT_ID")
	private Product product;

}
```

- Member.class

```java
public class Member{
	...

	//MEMBER_PRODUCT라는 중간 연결 테이블을 통해 풀어낸다
	@OneToMany(mappedBy = "member")
	private List<MemberProduct> memberProducts = new ArrayList<>();

	//Getter , Setter
```

- Product.class

```java
@Entity
public class Product{
	@Id @GeneratedValue
	private Long id;
	private String name;

	@OneToMany(mappedBy = "product")
	private List<MemberProduct> memberProducts = new ArrayList()<>;
}
```