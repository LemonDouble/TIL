# Spring Boot & JPA WebApp 개발 - 상품 도메인 개발

분류: SPRING+JPA
작성일시: 2021년 8월 21일 오후 1:33

## 1. 상품 엔티티 개발

- 구현 기능
    - 상품 등록
    - 상품 목록 조회
    - 상품 수정

- 순서
    - 상품 엔티티 개발(비즈니스 로직 추가)
    - 상품 리포지토리 개발
    - 상품 서비스 개발

- jpashop.domain.item.class 수정

```java
package jpabook.jpashop.domain.Item;

import jpabook.jpashop.domain.Category;
import jpabook.jpashop.exception.NotEnoughStockException;
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

    // ==비즈니스 로직 ==//
    // OOP 설계상, 핵심 비즈니스 로직은 클래스에 있는게 도움이 된다.
    // Setter를 열어놓고, 외부에서 바꾸지 말고, class 내부에 핵심 비즈니스 로직들을 구현하고
    // 외부에선 비즈니스 로직들을 call 하는 방식으로 구현하자.

    /*
    * stock 증가
     */

    public void addStock(int quantity){
        this.stockQuantity += quantity;
    }

    /*
    * stock 감소,
    * 단, stock은 0보다 작을 수 없음!
    */
    public void removeStock(int quantity){
        int restStock = this.stockQuantity - quantity;

        if(restStock < 0){
            throw new NotEnoughStockException("need more stock");
        }

        this.stockQuantity = restStock;
    }
}
```

## 2. 상품 리포지토리 개발

- jpashop.repositoy.ItemRepository.class

```java
package jpabook.jpashop.repository;

import jpabook.jpashop.domain.Item.Item;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Repository;

import javax.persistence.EntityManager;
import java.util.List;

@Repository
@RequiredArgsConstructor
public class ItemRepository {

    private final EntityManager em;

    public void save(Item item){
        //ID 없음 == 새로 생성한 객체, 따라서 em.persist로 신규로 등록
        if(item.getId() == null){
            em.persist(item);
        }else{
            //ID 있음 : 이미 DB에 등록되어 있음. 따라서 em.merge()로 업데이트
            em.merge(item);
        }
    }

    public Item findOne(Long id){
        return em.find(Item.class, id);
    }

    public List<Item> findAll(){
        return em.createQuery("select i from Item i", Item.class).getResultList();
    }
}
```

## 3. 상품 서비스 개발

단순히 위임만 하는 클래스!

- jpashop.service.itemService

```java
package jpabook.jpashop.service;

import jpabook.jpashop.domain.Item.Item;
import jpabook.jpashop.repository.ItemRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;

@Service
@Transactional(readOnly = true)
@RequiredArgsConstructor
public class ItemService {

    private final ItemRepository itemRepository;

    @Transactional
    public void saveItem(Item item){
        itemRepository.save(item);
    }

    public List<Item> findItems(){
        return itemRepository.findAll();
    }

    public Item findOne(Long itemId){
        return itemRepository.findOne(itemId);
    }
}
```