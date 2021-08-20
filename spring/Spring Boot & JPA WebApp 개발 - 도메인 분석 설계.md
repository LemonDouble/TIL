# Spring Boot & JPA WebApp 개발 - 도메인 분석 설계

분류: SPRING+JPA
작성일시: 2021년 7월 22일 오후 5:10

## 1. 요구사항 분석

- 예시 : 간단한 쇼핑몰 사이트

- 기능 목록
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
    - 기타 요구사항
        - 상품은 재고 관리가 필요하다
        - 상품의 종류는 도서, 음반, 영화가 있다
        - 상품은 카테고리로 구분할 수 있다

## 2. 도메인 모델과 테이블 설계

![Spring%20Boot%20&%20JPA%20WebApp%20%E1%84%80%E1%85%A2%E1%84%87%E1%85%A1%E1%86%AF%20-%20%E1%84%83%E1%85%A9%E1%84%86%E1%85%A6%E1%84%8B%E1%85%B5%E1%86%AB%20%E1%84%87%E1%85%AE%E1%86%AB%E1%84%89%E1%85%A5%E1%86%A8%20%E1%84%89%E1%85%A5%2088f949dd1a504d6a990a91702abb14b5/Untitled.png](Spring%20Boot%20&%20JPA%20WebApp%20%E1%84%80%E1%85%A2%E1%84%87%E1%85%A1%E1%86%AF%20-%20%E1%84%83%E1%85%A9%E1%84%86%E1%85%A6%E1%84%8B%E1%85%B5%E1%86%AB%20%E1%84%87%E1%85%AE%E1%86%AB%E1%84%89%E1%85%A5%E1%86%A8%20%E1%84%89%E1%85%A5%2088f949dd1a504d6a990a91702abb14b5/Untitled.png)

- 회원은 여러 상품을 주문할 수 있으니, 회원 : 주문 → 일대다!
- 회원이 한번 주문할 때, 여러 개의 상품을 주문할 수 있고, 상품도 여러 주문에 담길 수 있으므로 주문과 상품은 다대다 관계이다.
- 하지만 이런 다대다 관계는 DB는 물론이고 엔티티에서도 거의 사용하지 않는다.
- 따라서 중간에 주문상품이란 Entitiy 추가해 다대다 관계를 일대다, 다대일 관계로 풀어냈다.

- 상품은 도서, 음반, 영화로 구분되는데, 상품이라는 공통 속성을 사용하므로 상속 구조로 표현했다.

![Untitled](Spring%20Boot%20&%20JPA%20WebApp%20%E1%84%80%E1%85%A2%E1%84%87%E1%85%A1%E1%86%AF%20-%20%E1%84%83%E1%85%A9%E1%84%86%E1%85%A6%E1%84%8B%E1%85%B5%E1%86%AB%20%E1%84%87%E1%85%AE%E1%86%AB%E1%84%89%E1%85%A5%E1%86%A8%20%E1%84%89%E1%85%A5%2088f949dd1a504d6a990a91702abb14b5/Untitled%201.png)

- Category, Item도 OrderItem처럼 일대다, 다대일로 풀어내야 하나, ManyToMany 예시를 위하여 다대다를 그대로 사용했다.
    - 실제로 설계할땐 사용하면 안 된다!
- Item : Single Table Mapping 전략 사용해 Field가 많다 . DTYPE으로 타입을 구분한다.

- 연관관계 매핑 분석 ( 뒤에 대문자로 쓰인건 DB의 Table, 앞에껀 JPA의 변수)
    - 회원- 주문 : 일대다, 다대일 관계. 연관관계의 주인 정해야 하는데, FK 있는 주문(ORDER)을 연관관계의 주인으로 설정한다. 따라서 Order.member를 ORDERS.MEMBER_ID FK와 매핑한다.
    - 주문상품-주문 : 다대일 양방향 관계, FK가 주문상품(ORDER_ID) 에 있으므로, 주문상품이 연관관계의 주인이다. 따라서 OrderItem.order를 ORDER_ITEM.ORDER_ID FK와 매핑한다.
    - 주문상품-상품 : 다대일 단방향 관계, OrderItem.item을 ORDER_ITEM.ITEM_ID FK와 매핑
        - 왜 단방향? : 샴푸를 볼 때, 이 샴푸를 주문한 모든 주문내역 확인을 할 일이 없으니까!
    - 주문-배송 : 일대일 양방향 관계, Order.delivery를 ORDERS.DELIVERY_ID와 매핑
        - 일대일 관계인 경우에, FK를 어디에 둬도 상관없지만 예제와 같은 경우는 좀 더 많이 조회되는 쪽에 두었다.
    - 카테고리-상품 : @ManyToMany를 사용해서 매핑 (실제로는 사용하면 안됨!)

※ 연관관계의 주인은 항상 FK가 있는 곳으로 정하자!!! FK는 일대다에서 항상 다 쪽에 있다!

## 3. 엔티티 클래스 개발

- 예제에서는 쉽게 하기 위해 Getter, Setter 전부 열고 최대한 단순하게 설계
- 하지만 실제로는, Getter는 열어두고 Setter는 꼭 필요한 경우에만 사용하는 것을 추천

```
package jpabook.jpashop.domain;

import lombok.Getter;

import javax.persistence.Embeddable;

@Embeddable
@Getter
public class Address{

private String city;
    private String street;
    private String zipcode;

    protected Address() {
    }

public Address(Stringcity, Stringstreet, Stringzipcode) {
this.city =city;
        this.street =street;
        this.zipcode =zipcode;
}
}
```

- 폴더 구조

![Untitled](Spring%20Boot%20&%20JPA%20WebApp%20%E1%84%80%E1%85%A2%E1%84%87%E1%85%A1%E1%86%AF%20-%20%E1%84%83%E1%85%A9%E1%84%86%E1%85%A6%E1%84%8B%E1%85%B5%E1%86%AB%20%E1%84%87%E1%85%AE%E1%86%AB%E1%84%89%E1%85%A5%E1%86%A8%20%E1%84%89%E1%85%A5%2088f949dd1a504d6a990a91702abb14b5/Untitled%202.png)

- Album.class

```java
package jpabook.jpashop.domain.Item;

import lombok.Getter;
import lombok.Setter;

import javax.persistence.DiscriminatorValue;
import javax.persistence.Entity;

@Entity
//Dtype에서 "A" 로 들어간다.
@DiscriminatorValue("A")
@Getter @Setter
public class Album extends Item{
    private String artist;
    private String etc;
}
```

- Book.class

```java
package jpabook.jpashop.domain.Item;

import lombok.Getter;
import lombok.Setter;

import javax.persistence.DiscriminatorValue;
import javax.persistence.Entity;

//Dtype에서 "B" 로 들어간다.
@Entity
@DiscriminatorValue("B")
@Getter @Setter
public class Book extends Item{

    private String author;
    private String isbn;

}
```

- Item.class

```java
package jpabook.jpashop.domain.Item;

import jpabook.jpashop.domain.Category;
import lombok.Getter;
import lombok.Setter;

import javax.persistence.*;
import java.util.ArrayList;
import java.util.List;

@Entity
//상속 전략 (싱글 테이블 전략) 은 부모에 선언
@Inheritance(strategy=InheritanceType.SINGLE_TABLE)
//자식 클래스의 Data Type을 지정하는 컬럼 이름은 dtype
@DiscriminatorColumn(name="dtype")
@Getter @Setter
//구현체를 사용할 것이기 때문에 추상 클래스로 만든다
public abstract class Item {

    @Id @GeneratedValue
    @Column(name="item_id")
    private Long id;

    //공동 속성
    private String name;
    private int price;
    private int stockQuantity;

    @ManyToMany(mappedBy="items")
    private List<Category> categories = new ArrayList<>();
}
```

- Movie.class

```java
package jpabook.jpashop.domain.Item;

import lombok.Getter;
import lombok.Setter;

import javax.persistence.DiscriminatorValue;
import javax.persistence.Entity;

@Entity
@DiscriminatorValue("M")
@Getter @Setter
public class Movie extends Item{
    private String director;
    private String actor;
}
```

- Address.class

```java
package jpabook.jpashop.domain;

import lombok.Getter;

import javax.persistence.Embeddable;

@Embeddable
@Getter
public class Address {

    private String city;
    private String street;
    private String zipcode;

	//실제로 사용하진 않지만, JPA 스펙상 기본 생성자가 꼭 필요하다!
	//public, protected가 허용되는데, protected로 만들어서
	//이 생성자는 Spec상 만들어 둔 것이라는걸 알려준다.
    protected Address() {
    }

    public Address(String city, String street, String zipcode) {
        this.city = city;
        this.street = street;
        this.zipcode = zipcode;
    }
}
```

- Category.class

```java
package jpabook.jpashop.domain;

import jpabook.jpashop.domain.Item.Item;
import lombok.Getter;
import lombok.Setter;

import javax.persistence.*;
import java.util.ArrayList;
import java.util.List;

@Entity
@Getter @Setter
public class Category {

    @Id @GeneratedValue
    @Column(name="category_id")
    private Long id;

    private String name;

    //중간 Table에 추가로 데이터를 추가할 수 없으니 가급적 사용하지 말자!
    @ManyToMany
    @JoinTable(name="category_item",
            joinColumns = @JoinColumn(name="category_id"),
            inverseJoinColumns = @JoinColumn(name="item_id"))
    private List<Item> items = new ArrayList<>();

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "parent_id")
    private Category parent;

    @OneToMany(mappedBy ="parent")
    private List<Category> child = new ArrayList<>();

		/* 연관관계 편의 메서드 */

    public void addChildCategory(Category child){
        this.child.add(child);
        child.setParent(this);
    }

}
```

- Delivery.class

```java
package jpabook.jpashop.domain;

import lombok.Getter;
import lombok.Setter;

import javax.persistence.*;

@Entity
@Getter @Setter
public class Delivery {
    @Id @GeneratedValue
    @Column(name = "delivery_id")
    private Long id;

    @OneToOne(mappedBy="delivery", fetch = FetchType.LAZY)
    private Order order;

    @Embedded
    private Address address;

    //기본 설정은 EnumType.ORDINAL (숫자로 들어간다.)
    //하지만 나중에 Enum 수정시 숫자가 밀리거나, 뒤죽박죽 될 수 있다!
    //그러니 항상 STRING 사용하도록 하자.
    @Enumerated(EnumType.STRING)
    private DeliveryStatus status; //READY, COMP
}
```

- DeliveryStatus.enum

```java
package jpabook.jpashop.domain;

public enum DeliveryStatus {
    READY, COMP
}
```

- Member.class

```java
package jpabook.jpashop.domain;

import lombok.Getter;
import lombok.Setter;

import javax.persistence.*;
import java.util.ArrayList;
import java.util.List;

@Entity
@Getter @Setter
public class Member {

    @Id @GeneratedValue
    @Column(name="member_id")
    private Long id;
    private String name;

    @Embedded
    private Address address;

    @OneToMany(mappedBy="member")
    private List<Order> orders = new ArrayList<>();

}
```

- Order.class

```java
package jpabook.jpashop.domain;

import lombok.Getter;
import lombok.Setter;

import javax.persistence.*;
import java.time.LocalDateTime;
import java.util.ArrayList;
import java.util.List;

@Entity
@Table(name = "orders")
@Getter @Setter
public class Order {

    @Id @GeneratedValue
    @Column(name = "order_id")
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "member_id")
    private Member member;

		//
    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL)
    private List<OrderItem> orderItems = new ArrayList<>();

    @OneToOne(fetch = FetchType.LAZY , cascade = CascadeType.ALL)
    @JoinColumn(name = "delivery_id")
    private Delivery delivery;

    //Date 말고 LocalDateTime을 쓰자.
    //Date 쓰면 일일히 Mapping 해 줘야 하지만, LocalDateTime 쓰면 Hibernate가 매핑 해 준다.
    private LocalDateTime orderDate;

    @Enumerated(EnumType.STRING)
    private OrderStatus status;; //주문상태, [ORDER, CANCEL]

		/* 연관관계 편의 메서드*/

    public void setMember(Member member){
        this.member = member;
        member.getOrders().add(this);
    }
    /* 메서드 없다면 다음과 같이 작성해야 한다
        Member member = new Member();
        Order order = new Order();

        member.getOrders().add(order);
        order.setMember(member);
     */

    public void addOrderItem(OrderItem orderItem){
        orderItems.add(orderItem);
        orderItem.setOrder(this);
    }

    public void setDelivery(Delivery delivery){
        this.delivery = delivery;
        delivery.setOrder(this);
    }
}
```

- OrderItem.class

```java
package jpabook.jpashop.domain;

import jpabook.jpashop.domain.Item.Item;
import lombok.Getter;
import lombok.Setter;

import javax.persistence.*;

@Entity
@Table(name="order_item")
@Getter @Setter
public class OrderItem {

    @Id @GeneratedValue
    @Column(name="order_item_id")
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name="item_id")
    private Item item;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name="order_id")
    private Order order;

    private int orderPrice; //주문가격
    private int count; //주문 수량
}
```

- OrderStatus.enum

```java
package jpabook.jpashop.domain;

public enum OrderStatus {
    ORDER, CANCEL
}
```

## 4. 엔티티 설계시 주의점

- 엔티티에는 가급적 Setter를 닫아둔다.
    - Setter가 열려있으면 변경 포인트가 많아진다. 유지보수가 어렵다.
    - 별개의 메소드를 제작하여 사용하자.

- 모든 연관관계는 지연로딩으로 설정
    - 즉시 로딩(EAGER)는 예측이 어렵고, 어떤 SQL이 실행될지 추적하기 어렵다.
    - 특히, JPQL 실행시 N+1 문제가 자주 발생한다.
    - 따라서 모든 연관관계는 지연 로딩(LAZY)로 설정해야 한다.
    - 연관된 엔티티를 함꼐 DB에서 조회해야 하면, Fetch Join 혹은 엔티티 그래프 기능을 사용한다.
    - ~~ToOne (OneToOne, ManyToOne)은 기본이 즉시 로딩(EAGER)이므로 직접 지연로딩(LAZY)으로 설정해야 한다.

- 컬렉션은 필드에서 초기화하자.
    - 컬렉션은 필드에서 바로 초기화하는 것이 안전하다.
    - null 문제에서 안전한다.
    - 하이버네이티는 엔티티를 영속화할 때, 컬렉션을 감싸 하이버네이트가 제공하는 내장 컬렉션으로 변경한다. 만약, getOrders() 처럼 임의의 메서드에서 컬렉션을 잘못 생선하면 하이버네이트 내부 매커니즘에 문제가 발생할 수 있다. 따라서, 필드레벨에서 생성하는 것이 가장 안전하고 코드도 간결해진다.

    ```java
    Member member = new Member();
    System.out.println(member.getOrders().getClass());
    em.persist(team);
    System.out.println(member.getOrders().getClass());

    //출력 결과
    class java.util.ArrayList
    class org.hibernate.collection.internal.PersistentBag
    ```

- 테이블, 컬럼명 생성 전략
    - 스프링 부트에서 하이버네이트 기본 매핑 전략을 변경해서, 실제 테이블 필드명은 다름
        - 하이버네이트 기존 구현 : 엔티티의 필드명을 그대로 테이블명으로 사용
        - 스프링 부트 신규 설정
            1. 카멜 케이스 → 언더스코어 변경 (memberPoint → member_point)
            2. .(점) → _(언더스코어)
            3. 대문자→소문자