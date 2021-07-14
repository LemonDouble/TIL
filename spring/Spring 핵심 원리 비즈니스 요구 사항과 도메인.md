# Spring 핵심 원리 : 비즈니스 요구 사항과 도메인 설계

분류: Spring
작성일시: 2021년 7월 14일 오후 2:41

## 1. 비즈니스 요구사항과 설계

- 회원
    - 회원을 가입하고 조회할 수 있다.
    - 회원은 일반과 VIP, 두 등급이 있다.
    - 회원 데이터는 자체 DB를 구축할수도, 외부 시스템과 연결할 수도 있다 (미확정)
- 주문과 할인 정책
    - 회원은 상품을 주문할 수 있다.
    - 회원 등급에 따라 할인 정책을 변경할 수 있다.
    - 할인 정책은 모든 VIP는 1000원을 할인해주는 고정 금액 할인을 적용할 수 있다. (이후 변경이 가능하다)
    - 할인 정책은 변경 가능성이 높다. 기본 할인 정책은 아직 정해지지 않았고, 오픈 직전까지 고민을 미루려고 한다. 최악의 경우, 할인을 적용하지 않을 수도 있다.

→ 미확정인 부분이 많다. 이런 경우는 인터페이스를 만들고, 구현체를 바꿀 수 있도록 설계하면 된다.

## 2. 회원 도메인 설계

![Spring%20%E1%84%92%E1%85%A2%E1%86%A8%E1%84%89%E1%85%B5%E1%86%B7%20%E1%84%8B%E1%85%AF%E1%86%AB%E1%84%85%E1%85%B5%20%E1%84%87%E1%85%B5%E1%84%8C%E1%85%B3%E1%84%82%E1%85%B5%E1%84%89%E1%85%B3%20%E1%84%8B%E1%85%AD%E1%84%80%E1%85%AE%20%E1%84%89%E1%85%A1%E1%84%92%E1%85%A1%E1%86%BC%E1%84%80%E1%85%AA%20%E1%84%83%E1%85%A9%E1%84%86%E1%85%A6%E1%84%8B%E1%85%B5%E1%86%AB%20%202fe29b21d17a4427ba6e3f23c7cfe7e5/Untitled.png](https://github.com/LemonDouble/TIL/blob/main/spring/img/Untitled%205.png)

- MemoryMemberRepository, DbMemberRepository는 실행시간에 동적으로 설정됨

관례 : 구현체가 하나만 있는 경우 인터페이스 이름 뒤에 Impl로 많이 쓴다.

## 3. 회원 도메인 구현 (간단)

1. CoreApplication 있는 부분에, member Package 생성
2. Enum Grade 생성

    ```java
    package hello.core.member;

    public enum Grade {
        BASIC,
        VIP
    }
    ```

3. Class Member 생성 ( id, name, grade와 생성자, getter, setter)

```java
package hello.core.member;

public class Member {

    private Long id;
    private String name;
    private Grade grade;

    public Member(Long id, String name, Grade grade) {
        this.id = id;
        this.name = name;
        this.grade = grade;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Grade getGrade() {
        return grade;
    }

    public void setGrade(Grade grade) {
        this.grade = grade;
    }
}
```

4. MemberRepository 인터페이스 생성 (저장, findById)

```java
package hello.core.member;

public interface MemberRepository {

    void save(Member member);

    Member findById(Long memberId);
}
```

5. MemoryMemberRepository 클래스 (구현체) 생성

```java
package hello.core.member;

import java.util.HashMap;
import java.util.Map;

public class MemoryMemberRepository implements MemberRepository{

    private static Map<Long, Member> store = new HashMap<>();

    @Override
    public void save(Member member) {
        store.put(member.getId(), member);
    }

    @Override
    public Member findById(Long memberId) {
        return store.get(memberId);
    }
```

6. MemberService 인터페이스 생성 ( 회원가입, FindMember)

```java
package hello.core.member;

public interface MemberService {

    void join(Member member);

    Member findMember(Long memberId);
}
```

7. MemberServiceImpl 클래스 (구현체) 생성

```java
package hello.core.member;

public class MemberServiceImpl implements MemberService{

    private final MemberRepository memberRepository = new MemoryMemberRepository();

    @Override
    public void join(Member member) {
        memberRepository.save(member);
    }

    @Override
    public Member findMember(Long memberId) {
        return memberRepository.findById(memberId);
    }
}
```

이때, MemberServiceImpl의 문제점 :

- OCP, DIP 위반 : 의존관계가 인터페이스 뿐만 아니라 구현까지 모두 의존하는 문제점 있음.

## 4. 회원 도메인 실행과 테스트

1. 순수한 자바 코드로만 테스트 만들기 ⇒ CoreApplication 옆에 MemberApp.class 생성

```java
package hello.core;

import hello.core.member.Grade;
import hello.core.member.Member;
import hello.core.member.MemberService;
import hello.core.member.MemberServiceImpl;

public class MemberApp {
    public static void main(String[] args) {
        MemberService memberService = new MemberServiceImpl();
        Member member = new Member (1L, "memberA", Grade.VIP);
        memberService.join(member);

        Member findMember = memberService.findMember(1L);
        System.out.println("findMember = "+ member.getName());
				//findMember = memberA
        System.out.println("findMember = " + findMember.getName());
				//findMember = memberA
    }
}
```

2. Junit 사용하기 ⇒ test/java/hello.core/member/MemberServiceTest

```java
package hello.core.member;

import org.assertj.core.api.Assertions;
import org.junit.jupiter.api.Test;

public class MemberServiceTest {

    MemberService memberService = new MemberServiceImpl();

    @Test
    void join(){
        //given (~게 주어졌을 때)
        Member member = new Member(1L, "memberA", Grade.VIP);

        //when (~할 때)
        memberService.join(member);
        Member findMember = memberService.findMember(1L);

        //then (~하게 된다)
        Assertions.assertThat(member).isEqualTo(findMember);
        //org.assertj.core.api.Assertions
        //member랑 findMember이랑 같은가?
    }
}
```

## 5. 주문과 할인 도메인 설계

- 주문과 할인 정책
    - 회원은 상품을 주문할 수 있다.
    - 회원 등급에 따라 할인 정책을 변경할 수 있다.
    - 할인 정책은 모든 VIP는 1000원을 할인해주는 고정 금액 할인을 적용할 수 있다. (이후 변경이 가능하다)
    - 할인 정책은 변경 가능성이 높다. 기본 할인 정책은 아직 정해지지 않았고, 오픈 직전까지 고민을 미루려고 한다. 최악의 경우, 할인을 적용하지 않을 수도 있다.

![Spring%20%E1%84%92%E1%85%A2%E1%86%A8%E1%84%89%E1%85%B5%E1%86%B7%20%E1%84%8B%E1%85%AF%E1%86%AB%E1%84%85%E1%85%B5%20%E1%84%87%E1%85%B5%E1%84%8C%E1%85%B3%E1%84%82%E1%85%B5%E1%84%89%E1%85%B3%20%E1%84%8B%E1%85%AD%E1%84%80%E1%85%AE%20%E1%84%89%E1%85%A1%E1%84%92%E1%85%A1%E1%86%BC%E1%84%80%E1%85%AA%20%E1%84%83%E1%85%A9%E1%84%86%E1%85%A6%E1%84%8B%E1%85%B5%E1%86%AB%20%202fe29b21d17a4427ba6e3f23c7cfe7e5/Untitled%201.png](https://github.com/LemonDouble/TIL/blob/main/spring/img/Untitled%206.png)

- 예제를 간단하게 하기 위해, 주문을 DB에 저장하는 내용은 생략한다.
- 역할들의 협력 관계를 유지할 수 있다 == 회원 저장소가 메모리로 바뀌던 DB로 바뀌던, 주문 서비스 구현체는 바뀌지 않는다.

## 6. 주문과 할인 도메인 개발

1. discount Package 생성
2. DiscountPoilicy Interface 생성

```java
package hello.core.discount;

import hello.core.member.Member;

public interface DiscountPolicy {

    /**
     * @return 할인 대상 금액
    */
    int discount(Member member, int price);

}
```

3. 실제 구현체인 FixDiscountPolicy ( 고정 할인) Class 생성

```java
package hello.core.discount;

import hello.core.member.Grade;
import hello.core.member.Member;

public class FixDiscountPolicy implements DiscountPolicy{

    private int discountFixAmount = 1000;

    @Override
    public int discount(Member member, int price) {
        //VIP면 할인 가격만큼 할인, VIP가 아니면 0원 할인
        if(member.getGrade() == Grade.VIP){
            return discountFixAmount;
        }else{
            return 0;
        }
    }
}
```

4. order 패키지 생성

5. order 패키지 내 Order Model 생성

(memberId, itemName, itemPrice, discountPrice와 생성자, Getter, Setter,

그리고 추가로 비즈니스 로직을 위한 최종 가격 계산 calculatePrice() 를 추가했다)

```java
package hello.core.order;

public class Order {

    private Long memberId;
    private String itemName;
    private int itemPrice;
    private int discountPrice;

    public Order(Long memberId, String itemName, int itemPrice, int discountPrice) {
        this.memberId = memberId;
        this.itemName = itemName;
        this.itemPrice = itemPrice;
        this.discountPrice = discountPrice;
    }

    //비즈니스 로직
    public int calculatePrice(){
        //원가 - 할인가 계산
        return itemPrice - discountPrice;
    }

    public Long getMemberId() {
        return memberId;
    }

    public void setMemberId(Long memberId) {
        this.memberId = memberId;
    }

    public String getItemName() {
        return itemName;
    }

    public void setItemName(String itemName) {
        this.itemName = itemName;
    }

    public int getItemPrice() {
        return itemPrice;
    }

    public void setItemPrice(int itemPrice) {
        this.itemPrice = itemPrice;
    }

    public int getDiscountPrice() {
        return discountPrice;
    }

    public void setDiscountPrice(int discountPrice) {
        this.discountPrice = discountPrice;
    }

    @Override
    public String toString() {
        return "Order{" +
                "memberId=" + memberId +
                ", itemName='" + itemName + '\'' +
                ", itemPrice=" + itemPrice +
                ", discountPrice=" + discountPrice +
                '}';
    }
}
```

6. 그리고 주문에 대한 OrderService 인터페이스를 구현한다.

```java
package hello.core.order;

public interface OrderService {
    Order createOrder(Long memberId, String itemName, int itemPrice);

}
```

7. 이후, 실제 주문을 처리할 OrderSericeImpl 객체를 구현한다.

```java
package hello.core.order;

import hello.core.discount.DiscountPolicy;
import hello.core.discount.FixDiscountPolicy;
import hello.core.member.Member;
import hello.core.member.MemberRepository;
import hello.core.member.MemoryMemberRepository;

public class OrderServiceImpl implements OrderService {

    private final MemberRepository memberRepository = new MemoryMemberRepository();
    private final DiscountPolicy discountPolicy = new FixDiscountPolicy();

    @Override
    public Order createOrder(Long memberId, String itemName, int itemPrice) {
        Member member = memberRepository.findById(memberId);

        //잘 설계된 예시, OrderService는 Discount에 대해 관여하지 않는다.
        //단일 책임 원칙 (SRP: Single Responsibility Principle) 이 잘 지켜진 예시
        int discountPrice = discountPolicy.discount(member,itemPrice);

        return new Order(memberId, itemName, itemPrice, discountPrice);
    }
}
```

## 7. 주문과 할인 도메인 실행과 테스트

1.orderApp.class로 Test (좋지 않은 방법)

```java
package hello.core.order;

import hello.core.member.Grade;
import hello.core.member.Member;
import hello.core.member.MemberService;
import hello.core.member.MemberServiceImpl;

public class OrderApp {
    public static void main(String[] args) {
        MemberService memberService = new MemberServiceImpl();
        OrderService orderService = new OrderServiceImpl();

        Long memberId = 1L;
        Member member = new Member(memberId, "memberA", Grade.VIP);
        memberService.join(member);

        Order order = orderService.createOrder(memberId, "itemA", 10000);

        System.out.println("order = " + order);
    }
}
```

2. test/order/OrderServiceTest (Junit 사용)

```java
package hello.core.order;

import hello.core.member.Grade;
import hello.core.member.Member;
import hello.core.member.MemberService;
import hello.core.member.MemberServiceImpl;
import org.assertj.core.api.Assertions;
import org.junit.jupiter.api.Test;

public class OrderServiceTest {

    MemberService memberService = new MemberServiceImpl();
    OrderService orderService = new OrderServiceImpl();

    @Test
    void createOrder(){
        Long memberId = 1L;
        Member member = new Member(memberId, "memberA", Grade.VIP);
        memberService.join(member);

        Order order = orderService.createOrder(memberId, "itemA", 10000);
        Assertions.assertThat(order.getDiscountPrice()).isEqualTo(1000);
    }
}
```
