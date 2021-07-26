# 자바 ORM 표준 JPA 프로그래밍 - 고급 매핑 (상속관계 매핑)

분류:  JPA
작성일시: 2021년 7월 26일 오후 5:31

## 1. 상속관계 매핑

- 객체 → 상속관계 있음
- RDB → 상속관계 없음
- RDB의 SuperType - SubType이라는 모델링 기법이 객체 상속과 유사
- 상속관계 매핑 : 색체의 상속 구조와 DB의 슈퍼타입-서브타임 관계를 매핑

- 슈퍼타입/서브타입 논리 모델을 실제 물리 모델로 구현하는 방법
    1. 각각 테이블로 변환 → Join 전략
        - 슈퍼타입은 공통인자와, Data Type 가진다
        - 각 Data Type은 해당 Data Type에 맞는 데이터들 가진다
        - 두 자료형 JOIN해서 돌려준다!
            - 예시
                - 슈퍼타입 : ID(PK), 가격, 이름, 데이터타입 (앨범, 책)
                - 서브타입1(앨범) : ITEM_ID (PK, FK), 아티스트
                - 서브타입2(책) : ITEM_ID(PK,FK), 저자, ISBN
    2. 통합 테이블로 변환 → 단일 테이블 전략
        - 하나의 테이블에 모든 Feature Column 다 만든다!
        - 예시
            - ID (PK), 가격, 이름, 아티스트, 저자, ISBN 데이터타입 (앨범, 책)
    3. 서브타입 테이블로 변환 → 구현 클래스마다 테이블 전략 (비추천)
        - 테이블을 세개 만들어서, 각각 데이터를 가진다
        - 예시
            - 앨범 : ID(PK), 이름, 가격, 아티스트
            - 책 : ID(PK), 이름, 가격, 저자, ISBN

- JOIN 전략
    - 장점)
        - 테이블 정규화
        - 외래 키 참조 무결성 제약요건 활용 가능
        - 저장공간 효율화
    - 단점)
        - 조회시 조인을 많이 사용, 성능 저하
        - 조회 쿼리가 복잡함
        - 데이터 저장시 INSERT SQL 두번 호출

- 단일 테이블 전략
    - 장점)
        - JOIN이 필요없으므로, 일반적으로 조회 성능이 빠름
        - 조회 쿼리가 단순함
    - 단점)
        - 자식 엔티티가 매핑한 컬럼은 모두 NULL 허용
        - 단일 테이블에 모든 것을 저장하므로, 테이블이 커질 수 있다.
        - 상황에 따라서, 역으로 조회 성능이 느려질 수도 있다.

- 구현 클래스마다 테이블 전략
    - 이 전략은 DB 설계자, ORM 전문가 둘 다 추천 X
    - 장점)
        - 서브타입을 명확하게 구분해서 처리할 떄 효과적
        - not null 제약조건 사용 가능
    - 단점)
        - 여러 자식 테이블을 함께 조회할 때 성능이 느림 (UNION SQL 필요)
        - 자식 테이블을 통합해서 쿼리하기 어려움 → 부모 클래스로 조회하기 힘들다!!

- Goods.class

```java
package hellojpa.domain;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

@Entity
//@Inheritance(strategy = InheritanceType.JOINED)
//JOIN 전략 사용, 만약 설정하지 않을 경우, 단일 테이블 전략으로 사용된다.
public class Goods {

    @Id @GeneratedValue
    private Long id;

    private String name;
    private int price;

}
```

- Album.class

```java
package hellojpa.domain;

import javax.persistence.Entity;

@Entity
public class Album extends Goods{
    private String artist;
}
```

- Book.class

```java
package hellojpa.domain;

import javax.persistence.Entity;

@Entity
public class Book extends Goods{
    private String author;
    private String isbn;
}
```

- Movie.class

```java
package hellojpa.domain;

import javax.persistence.Entity;

@Entity
public class Movie extends Goods{
    private String director;
    private String actor;
}
```

- 바로 실행시, 단일 테이블(Single Table) 전략으로 매칭된다

```java
create table Goods (
       DTYPE varchar(31) not null,
        id bigint not null,
        name varchar(255),
        price integer not null,
        artist varchar(255),
        author varchar(255),
        isbn varchar(255),
        actor varchar(255),
        director varchar(255),
        primary key (id)
    )
```

- Annotation
    - @Inheritance(strategy=InheritanceType.XXX)
        - JOINED : 조인 전략
        - SINGLE_TABLE : 단일 테이블 전략 (기본)
        - TABLE_PER_CLASS : 구현 클래스마다 테이블 전략
    - @DiscriminatorColumn(name=“DTYPE”)
        - 부모 클래스에서 사용
        - DTYPE이라는 Column 생긴다.
        - DTYPE 내부 : Album, Movie, Book 등..
        - name을 바꾸면 (예시 : DIS_TYPE) 컬럼명이 바뀐다.
        - DB 조회하면, 바로 자식 클래스 타입 알 수 있으므로 (명확하므로) 쓰는걸 권장
    - @DiscriminatorValue(“XXX”)
        - 자식 클래스에서 사용
        - XXX 안에 들어간 내용이, 부모 클래스의 DTYPE에 들어간다.
        - 만약 ALBUM에 @DiscriminatorValue(“A”) 어노테이션 넣는다면, DTYPE에 A 들어옴

- JOIN 전략을 기본으로 사용하되, 확장 가능성도 없고 아주 간단한 경우에만 단일 테이블 전략을 사용하자!

## 2. @Mapped Superclass - 매핑 정보 상속

![%E1%84%8C%E1%85%A1%E1%84%87%E1%85%A1%20ORM%20%E1%84%91%E1%85%AD%E1%84%8C%E1%85%AE%E1%86%AB%20JPA%20%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%80%E1%85%B3%E1%84%85%E1%85%A2%E1%84%86%E1%85%B5%E1%86%BC%20-%20%E1%84%80%E1%85%A9%E1%84%80%E1%85%B3%E1%86%B8%20%E1%84%86%E1%85%A2%E1%84%91%E1%85%B5%E1%86%BC%20(%E1%84%89%E1%85%A1%E1%86%BC%E1%84%89%20b4d0ca6e3b9d478cb20d347862d55409/Untitled.png](https://github.com/LemonDouble/TIL/blob/main/JPA/img/Untitled%2019.png)

- 객체의 입장에서, 항상 나오는 Field를 묶어서 관리하고 싶을 때!
- 예를 들어, DB 관리정책상 모든 데이터에 생성시간, 생성일, 마지막 수정시간, 마지막 수정일이 남아야 한다면?

- @MappedSuperclass
    - DB 등에 수정 없이, 단순히 공통적으로 사용할 Field가 있는 경우
    - 상속관계가 매핑되지 않는다.
    - 엔티티도 아니고, DB의 테이블과 매핑되지도 않는다.
    - 부모 클래스를 상속받는 자식 클래스에 매핑 정보만 제공한다.
    - 조회, 검색 불가(em.find(BaseEntity) 불가) → DB에 해당 Table 없으므로!
    - 직접 생성해서 사용할 일이 없으므로, 추상 클래스 권장!

    - 참고) @Entity 클래스는 @Entity나, @MappedSuperClass로 지정한 클래스만 상속 가능
- BaseEntity.class

```java
@MappedSuperclass
public class BaseEntity{
	private String createBy;
	private LocalDateTime createDate;
	private String lastModifiedBy;
	private LocalDateTime lastModifiedDate;
}
```

- 다른 클래스에서 상속받아 사용

```java
@Entity
public class Member extends BaseEntity{

}
```