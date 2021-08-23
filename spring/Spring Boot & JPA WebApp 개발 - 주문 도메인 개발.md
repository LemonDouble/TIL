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
package jpabook.jpashop.domain;

import lombok.Getter;
import lombok.Setter;

import javax.persistence.*;
import java.time.LocalDate;
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

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL)
    private List<OrderItem> orderItems = new ArrayList<>();

    @OneToOne(fetch = FetchType.LAZY, cascade = CascadeType.ALL)
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

    /* == 생성 메서드 == */
    // Order 를 만드는 것은 복잡하다. 연관관계 설정, Member, Delivery 등등..
    // 또한 생성 시점을 바꿔야 하는 경우, 이 함수의 위치만 바꾸면 된다.
    public static Order createOrder(Member member, Delivery delivery, OrderItem... orderItems){
        Order order = new Order();
        order.setMember(member);
        order.setDelivery(delivery);
        for(OrderItem orderItem : orderItems){
            order.addOrderItem(orderItem);
        }
        order.setStatus(OrderStatus.ORDER);
        order.setOrderDate(LocalDateTime.now());
        return order;
    }

    /* == 비즈니스 로직 == */

    /*
    * 주문 취소
     */
    public void cancel(){
        if(delivery.getStatus() == DeliveryStatus.COMP){
            throw new IllegalStateException("이미 배송완료된 상품은 취소가 불가능합니다.");
        }

        this.setStatus(OrderStatus.CANCEL);
        for (OrderItem orderItem : this.orderItems){
            orderItem.cancel();
        }
    }

    /* == 조회 로직 == */

    /*
    * 전체 주문 가격 조회
    */

    public int getTotalPrice(){

        /*
        int totalPrice = 0;
        for(OrderItem orderItem : this.orderItems){
            totalPrice += orderItem.getTotalPrice();
        }
        return totalPrice;
        */
        // 위와 같은 코드!
        return orderItems.stream().mapToInt(OrderItem::getTotalPrice).sum();
    }
}
```

- jpashop.domain.OrderItem.class 수정

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

    /* == 생성 메서드 ==*/
    public static OrderItem createOrderItem(Item item, int orderPrice, int count){
        OrderItem orderItem = new OrderItem();
        orderItem.setItem(item);
        orderItem.setOrderPrice(orderPrice);
        orderItem.setCount(count);

        item.removeStock(count);
        return orderItem;
    }

    /* == 비즈니스 로직 ==*/
    public void cancel(){
        getItem().addStock(count);
    }

    public int getTotalPrice() {
        return getOrderPrice() * getCount();
    }
}
```

## 3. 주문 리포지토리 개발

- jpashop.repository.OrderRepository.class

```java
package jpabook.jpashop.repository;

import jpabook.jpashop.domain.Order;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Repository;

import javax.persistence.EntityManager;
import java.util.List;

@Repository
@RequiredArgsConstructor
public class OrderRepository {
    private final EntityManager em;

    public void save(Order order){
        em.persist(order);
    }

    public Order findOne(Long id){
        return em.find(Order.class, id);
    }

    // 추후 개발
    // public List<Order> findAll(OrderSearch orderSearch) {}
}
```

## 4. 주문 서비스 개발

- jpashop.service.OrderService.class

```java
package jpabook.jpashop.service;

import jpabook.jpashop.domain.Delivery;
import jpabook.jpashop.domain.Item.Item;
import jpabook.jpashop.domain.Member;
import jpabook.jpashop.domain.Order;
import jpabook.jpashop.domain.OrderItem;
import jpabook.jpashop.repository.ItemRepository;
import jpabook.jpashop.repository.MemberRepository;
import jpabook.jpashop.repository.OrderRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;

@Service
@Transactional(readOnly = true)
@RequiredArgsConstructor
public class OrderService {

    private final OrderRepository orderRepository;
    private final MemberRepository memberRepository;
    private final ItemRepository itemRepository;
    /*
    * 주문
     */
    @Transactional
    public Long order(Long memberId, Long itemId, int count){
        //엔티티 조회
        Member member = memberRepository.findOne(memberId);
        Item item = itemRepository.findOne(itemId);

        //배송정보 생성
        //간단화된 예시이므로, 회원의 Address 그대로 쓴다고 가정.
        Delivery delivery = new Delivery();
        delivery.setAddress(member.getAddress());

        //주문상품 생성
        OrderItem orderItem = OrderItem.createOrderItem(item, item.getPrice(), count);

        //주문 생성
        Order order = Order.createOrder(member, delivery, orderItem);

        //주문 저장
        //orderItem, delivery는 save 안 했는데 save 되는 이유
        //order domain에 걸려 있는 CascadeType.all 때문.
        //order 저장할 때 orderItem, delivery도 같이 저장된다.

        // casacde는 언제 쓰나요?
        // 이 경우, delivery나 orderItem은 order만 참조하고 사용함.
        // 또한, 다른 객체는 참조하지 않으므로 cascade 사용할 수 있음.
        // 근데 만약 delivery를 다른 객체들에서 참조해서 쓰면 cascade 지워야 한다
        // 만약 order 지워질때 delivery도 지워지면???

        // 따라서
        // 나만 얘를 참조해서 쓰는 경우 -> cacade 사용 가능
        // 다른 객체도 쟤를 참조하는 경우 -> 별도의 Repository 만들어서, 명시적으로 save 해 주자.
        orderRepository.save(order);
        return order.getId();
    }

    /*
    * 주문 취소
     */
    @Transactional
    public void cancelOrder(Long orderId){
        //주문 엔티티 조회
        Order order = orderRepository.findOne(orderId);
        //주문 취소
        order.cancel();
    }

    //검색 (추후 개발)
    /*
    public List<Order> findOrders(OrderSearch orderSearch){
        return orderRepository.findAll(orderSearch);
    }
     */
}
```

- 지금의 경우, 도메인에 핵심 비즈니스 로직이 있고, 서비스 계층은 단순히 엔티티에 필요한 요청을 위임하는 역할만 하고 있다.
- 이처럼 엔티티가 비즈니스 로직을 가지고, 객체 지향의 특성을 확인하는 방법을 도메인 모델 패턴 ( [https://martinfowler.com/eaaCatalog/domainModel.html](https://martinfowler.com/eaaCatalog/domainModel.html) ) 라고 한다.

- 반대로, 엔티티에는 비즈니스 로직이 거의 없고, 서비스 계층에서 대부분의 비즈니스 로직을 처리하는 것을 트랜잭션 스크립트 패턴이라고 한다. ([https://martinfowler.com/eaaCatalog/transactionScript.html](https://martinfowler.com/eaaCatalog/transactionScript.html))
- Domain은 getter, setter밖에 없고, 서비스 단에서 직접 SQL 쿼리를 날리는 경우를 생각해 보자!

## 5. 주문 기능 테스트

- test/.../jpashop.service.OrderServiceTest.class

```java
package jpabook.jpashop.service;

import jpabook.jpashop.domain.Address;
import jpabook.jpashop.domain.Item.Book;
import jpabook.jpashop.domain.Item.Item;
import jpabook.jpashop.domain.Member;
import jpabook.jpashop.domain.Order;
import jpabook.jpashop.domain.OrderStatus;
import jpabook.jpashop.exception.NotEnoughStockException;
import jpabook.jpashop.repository.OrderRepository;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit.jupiter.SpringExtension;

import javax.persistence.EntityManager;
import javax.transaction.Transactional;

import static org.junit.jupiter.api.Assertions.*;

@ExtendWith(SpringExtension.class)
@SpringBootTest
@Transactional
class OrderServiceTest {

    @Autowired
    EntityManager em;
    @Autowired
    OrderService orderService;
    @Autowired
    OrderRepository orderRepository;

    @Test
    public void 상품주문() throws Exception {
        //given
        Member member = createMember();

        Item book = createBook("JPA", 10000, 10);

        //when
        int orderCount = 2;
        Long orderId = orderService.order(member.getId(), book.getId(), orderCount);

        //then
        Order getOrder = orderRepository.findOne(orderId);

        assertEquals(OrderStatus.ORDER, getOrder.getStatus(), "상품 주문시 상태는 order");
        assertEquals(1, getOrder.getOrderItems().size(),"주문한 상품 수(종류)가 정확해야 한다.");
        assertEquals(10000*orderCount , getOrder.getTotalPrice(), "주문 가격은 가격 * 수량이다");
        assertEquals(8,book.getStockQuantity(), "주문 수량만큼 재고가 줄어야 한다.");
    }

    @Test
    public void 상품주문_재고수량초과() throws Exception {
        //NotEnoughStockException 발생해야 한다!
        assertThrows(NotEnoughStockException.class, () -> {
            //given
            Member member = createMember();
            Item book = createBook("JPA", 10000, 10);

            int orderCount = 11;

            //when
            orderService.order(member.getId(), book.getId(), orderCount); //예외 발생해야 한다!

            //then
            fail("재고 수량 부족 예외가 발생해야 한다.");
        });
    }

    @Test
    public void 주문취소() throws Exception {
        //given
        Member member = createMember();
        Item book = createBook("JPA", 10000, 10);

        int orderCount = 2;
        Long orderId = orderService.order(member.getId(), book.getId(), orderCount);

        //when

        orderService.cancelOrder(orderId);

        //then
        Order getOrder = orderRepository.findOne(orderId);

        assertEquals(OrderStatus.CANCEL, getOrder.getStatus(), "주문 취소시 상태가 CANCEL로 바뀐다.");
        assertEquals(10, book.getStockQuantity(), "주문 취소시 상품은 그만큼 재고가 증가해야 한다.");
    }

    private Item createBook(String name, int price, int stockQuantity) {
        Item book = new Book();
        book.setName(name);
        book.setPrice(price);
        book.setStockQuantity(stockQuantity);
        em.persist(book);
        return book;
    }

    private Member createMember() {
        Member member = new Member();
        member.setName("회원1");
        member.setAddress(new Address("서울","강가","123-123"));
        em.persist(member);
        return member;
    }
}
```

## 6. 주문 검색 기능 개발

- 회원 이름, 주문상태 가지고 찾는거 만듬!

![Untitled](https://github.com/LemonDouble/TIL/blob/main/spring/img/Untitled%2026.png)

- jpashop.repository.OrderSearch.class

```java
package jpabook.jpashop.repository;

import jpabook.jpashop.domain.OrderStatus;
import lombok.Getter;
import lombok.Setter;

@Getter @Setter
public class OrderSearch {

    private String memberName; //회원 이름
    private OrderStatus orderStatus; //주문 상태 [ORDER, CANCEL]

    public OrderSearch(String memberName, OrderStatus orderStatus) {
        this.memberName = memberName;
        this.orderStatus = orderStatus;
    }
}
```

- OrderRepository.class 의 findAll() 함수 수정

```java
public List<Order> findAll(OrderSearch orderSearch) {
        return em.createQuery(" select o from Order o join o.member m " +
                " where o.status = :status " +
                " and m.name like :name ", Order.class)
                .setParameter("status", orderSearch.getOrderStatus())
                .setParameter("name", orderSearch.getMemberName())
                .setMaxResults(1000) //최대 1000개까지
                .getResultList();
    }
```

- OrderService.class 의 findOrders() 함수 수정

```java
public List<Order> findOrders(OrderSearch orderSearch){
        return orderRepository.findAll(orderSearch);
    }
```

- OrderServiceTest.class에서 새로 Test 해서 검증

```java
@Test
    public void 주문_검색() throws Exception {
        //given
        Member member = createMember("회원1");
        Member member2 = createMember("회원2");
        Item book1 = createBook("JPA1", 10000, 10);
        Item book2 = createBook("JPA2", 20000, 20);
        Item book3 = createBook("JPA3", 30000, 30);

        int orderCount = 1;
        Long orderId1 = orderService.order(member.getId(), book1.getId(), orderCount);
        Long orderId2 = orderService.order(member.getId(), book2.getId(), orderCount);
        Long orderId3 = orderService.order(member.getId(), book3.getId(), orderCount);
        Long orderId4 = orderService.order(member2.getId(), book1.getId(), orderCount);
        Long orderId5 = orderService.order(member2.getId(), book2.getId(), orderCount);
        Long orderId6 = orderService.order(member2.getId(), book3.getId(), orderCount);

        orderService.cancelOrder(orderId1);
        orderService.cancelOrder(orderId2);
        orderService.cancelOrder(orderId4);

        List<Order> correct_member1_canceledOrder = new ArrayList<>();
        correct_member1_canceledOrder.add(orderRepository.findOne(orderId1));
        correct_member1_canceledOrder.add(orderRepository.findOne(orderId2));

        List<Order> correct_member1_acceptedOrder = new ArrayList<>();
        correct_member1_acceptedOrder.add(orderRepository.findOne(orderId3));

        List<Order> correct_member2_canceledOrder = new ArrayList<>();
        correct_member2_canceledOrder.add(orderRepository.findOne(orderId4));

        List<Order> correct_member2_acceptedOrder = new ArrayList<>();
        correct_member2_acceptedOrder.add(orderRepository.findOne(orderId5));
        correct_member2_acceptedOrder.add(orderRepository.findOne(orderId6));

        //when

        List<Order> member1_canceledOrder = orderService.findOrders(new OrderSearch(member.getName(), OrderStatus.CANCEL));
        List<Order> member1_acceptedOrder = orderService.findOrders(new OrderSearch(member.getName(), OrderStatus.ORDER));

        List<Order> member2_canceledOrder = orderService.findOrders(new OrderSearch(member2.getName(), OrderStatus.CANCEL));
        List<Order> member2_acceptedOrder = orderService.findOrders(new OrderSearch(member2.getName(), OrderStatus.ORDER));
        //then

        assertEquals(correct_member1_canceledOrder, member1_canceledOrder);
        assertEquals(correct_member1_acceptedOrder, member1_acceptedOrder);
        assertEquals(correct_member2_canceledOrder, member2_canceledOrder);
        assertEquals(correct_member2_acceptedOrder, member2_acceptedOrder);
    }
```
