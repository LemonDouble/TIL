# Spring Boot & JPA WebApp 개발 - 웹 계층 개발

분류: SPRING+JPA
작성일시: 2021년 8월 23일 오후 7:57

## 1. 홈 화면과 레이아웃

- Controller Package 만듬
- jpashop.controller.HomeController.class

```java
package jpabook.jpashop.controller;

import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
//Slf4j 로거를 사용하기 위한 Annotation
@Slf4j
public class HomeController {

    @RequestMapping("/")
    public String home(){
        log.info("home controller");
        return "home";
    }
}
```

- resources.teamplates.home.html

```java
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragments/header :: header">
    <title>Hello</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>
<div class="container">
    <div th:replace="fragments/bodyHeader :: bodyHeader" />
    <div class="jumbotron">
        <h1>HELLO SHOP</h1>
        <p class="lead">회원 기능</p>
        <p>
            <a class="btn btn-lg btn-secondary" href="/members/new">회원 가입</a>
            <a class="btn btn-lg btn-secondary" href="/members">회원 목록</a>
        </p>
        <p class="lead">상품 기능</p>
        <p>
            <a class="btn btn-lg btn-dark" href="/items/new">상품 등록</a>
            <a class="btn btn-lg btn-dark" href="/items">상품 목록</a>
        </p>
        <p class="lead">주문 기능</p>
        <p>
            <a class="btn btn-lg btn-info" href="/order">상품 주문</a>
            <a class="btn btn-lg btn-info" href="/orders">주문 내역</a>
        </p>
    </div>
    <div th:replace="fragments/footer :: footer" />
</div> <!-- /container -->
</body>
</html>
```

- BootStrap 4.3.1 버전 받아서 static 폴더에 품.

  - resource → static → css / js 되도록

- templates.에 fragments 폴더 만들어서
- fragments/bodyheader.html

```java
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<div class="header" th:fragment="bodyHeader">
    <ul class="nav nav-pills pull-right">
        <li><a href="/">Home</a></li>
    </ul>
    <a href="/"><h3 class="text-muted">HELLO SHOP</h3></a>
</div>
```

- fragments/footer.html

```java
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<div class="footer" th:fragment="footer">
    <p>&copy; Hello Shop V2</p>
</div>
```

- fragments/header.html

```java
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head th:fragment="header">
    <!-- Required meta tags -->
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrinkto-
fit=no">
    <!-- Bootstrap CSS -->
    <link rel="stylesheet" href="/css/bootstrap.min.css" integrity="sha384-
ggOyR0iXCbMQv3Xipma34MD+dH/1fQ784/j6cY/iJTQUOhcWr7x9JvoRxT2MZw1T"
          crossorigin="anonymous">
    <!-- Custom styles for this template -->
    <link href="/css/jumbotron-narrow.css" rel="stylesheet">
    <title>Hello, world!</title>
</head>
```

- static/css/jumbotron-narrow.css 추가

```java
/* Space out content a bit */
body {
padding-top: 20px;
padding-bottom: 20px;
}
/* Everything but the jumbotron gets side spacing for mobile first views */
.header,
.marketing,
.footer {
padding-left: 15px;
padding-right: 15px;
}
/* Custom page header */
.header {
border-bottom: 1px solid #e5e5e5;
}
/* Make the masthead heading the same height as the navigation */
.header h3 {
margin-top: 0;
margin-bottom: 0;
line-height: 40px;
padding-bottom: 19px;
}
/* Custom page footer */
.footer {
padding-top: 19px;
color: #777;
border-top: 1px solid #e5e5e5;
}
/* Customize container */
@media (min-width: 768px) {
.container {
max-width: 730px;
}
}
.container-narrow > hr {
margin: 30px 0;
}
/* Main marketing message and sign up button */
.jumbotron {
text-align: center;
border-bottom: 1px solid #e5e5e5;
}
.jumbotron .btn {
font-size: 21px;
padding: 14px 24px;
}
/* Supporting marketing content */
.marketing {
margin: 40px 0;
}
.marketing p + h4 {
margin-top: 28px;
}
/* Responsive: Portrait tablets and up */
@media screen and (min-width: 768px) {
/* Remove the padding we set earlier */
.header,
.marketing,
.footer {
padding-left: 0;
padding-right: 0;
}
/* Space out the masthead */
.header {
margin-bottom: 30px;
}
/* Remove the bottom border on the jumbotron for visual effect */
.jumbotron {
border-bottom: 0;
}
}
```

## 2. 회원 등록

- jpashop.controller.MemberForm.class

```java
package jpabook.jpashop.controller;

import lombok.Getter;
import lombok.Setter;

import javax.validation.constraints.NotEmpty;

@Getter @Setter
public class MemberForm {

    //Annotation 통해서 에러 알려준다.
    @NotEmpty(message ="회원 이름은 필수 입니다.")
    private String name;

    private String city;
    private String street;
    private String zipcode;
}
```

- jpashop.controller.MemberController.class

```java
package jpabook.jpashop.controller;

import jpabook.jpashop.domain.Address;
import jpabook.jpashop.domain.Member;
import jpabook.jpashop.service.MemberService;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.validation.BindingResult;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;

import javax.validation.Valid;

@Controller
@RequiredArgsConstructor
public class MemberController {

    private final MemberService memberService;

    @GetMapping("/members/new")
    public String createForm(Model model){
        //addAttribute : View 화면에 데이터를 들고 갈 수 있게 해준다. memberForm class를 들고 간다.
        model.addAttribute("memberForm", new MemberForm());
        return "members/createMemberForm";
    }

    //@Valid : Validation (MemberForm의 @NotEmpty Annotation 을 활성화한다.)
    //BindingResult : 오류가 생기면 result가 실행된다.
    @PostMapping("/members/new")
    public String create(@Valid MemberForm form, BindingResult result){

        //result에 error 있다면 redirect 한다.
        if(result.hasErrors()){
            return "members/createMemberForm";
        }
        Address address = new Address(form.getCity(), form.getStreet(), form.getZipcode());

        Member member = new Member();
        member.setName(form.getName());
        member.setAddress(address);

        memberService.join(member);
        return "redirect:/";
    }
}
```

- resources/templates/member/createMemberForm.html

```java
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragments/header :: header" />
<style>
.fieldError {
border-color: #bd2130;
}
</style>
<body>
<div class="container">
    <div th:replace="fragments/bodyHeader :: bodyHeader"/>
		<!-- th:object 부분 : form의 데이터가 memberForm으로 받아진다. -->
    <form role="form" action="/members/new" th:object="${memberForm}" method="post">
        <div class="form-group">
            <label th:for="name">이름</label>
						<!-- th:field 부분 : id ="name" , name ="name"과 같다. 위에 th: object로 설정한 memberForm의 field와 매칭해줌.-->
            <input type="text" th:field="*{name}" class="form-control"
                   placeholder="이름을 입력하세요"
									<!-- 만약 controller에서 result.hasErrors() 분기 타고 왔다면, fieldError 클래스 추가되어
									Border가 빨간색으로 변경된다.-->
                   th:class="${#fields.hasErrors('name')}? 'form-control fieldError' : 'form-control'">
						<!-- 만약 feild.hasErrors() 분기 타고 왔다면
						@NotEmpty(message ="회원 이름은 필수 입니다.")
						설정한 에러 메세지를 보여준다.-->
            <p th:if="${#fields.hasErrors('name')}" th:errors="*{name}">Incorrect date</p>
        </div>
        <div class="form-group">
            <label th:for="city">도시</label>
            <input type="text" th:field="*{city}" class="form-control"
                   placeholder="도시를 입력하세요">
        </div>
        <div class="form-group">
            <label th:for="street">거리</label>
            <input type="text" th:field="*{street}" class="form-control"
                   placeholder="거리를 입력하세요">
        </div>
        <div class="form-group">
            <label th:for="zipcode">우편번호</label>
            <input type="text" th:field="*{zipcode}" class="form-control"
                   placeholder="우편번호를 입력하세요">
        </div>
        <button type="submit" class="btn btn-primary">Submit</button>
    </form>
    <br/>
    <div th:replace="fragments/footer :: footer" />
</div> <!-- /container -->
</body>
</html>
```

## 3. 회원 목록 조회

- MemberController에 추가

```java
@GetMapping("/members")
    public String list(Model model){
        List<Member> members = memberService.findMembers();
        model.addAttribute("members", members);
        return "members/memberList";
    }
```

- templates/members/memberList.html에 추가

```java
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragments/header :: header" />
<body>
<div class="container">
    <div th:replace="fragments/bodyHeader :: bodyHeader" />
    <div>
        <table class="table table-striped">
            <thead>
            <tr>
                <th>#</th>
                <th>이름</th>
                <th>도시</th>
                <th>주소</th>
                <th>우편번호</th>
            </tr>
            </thead>
            <tbody>
            <tr th:each="member : ${members}">
                <td th:text="${member.id}"></td>
                <td th:text="${member.name}"></td>
                <!-- ?를 붙이는 경우, null을 무시한다.
                이 경우 만약 Address가 Null이라면 출력되지 않는다.-->
                <td th:text="${member.address?.city}"></td>
                <td th:text="${member.address?.street}"></td>
                <td th:text="${member.address?.zipcode}"></td>
            </tr>
            </tbody>
        </table>
    </div>
    <div th:replace="fragments/footer :: footer" />
</div> <!-- /container -->
</body>
</html>
```

- Domain의 Entity를 사용하지 않고, Form 객체를 별도로 생성하는 이유

  1. Form과 Entity가 100% 매칭되는 경우는 (실제로는) 되게 드물다.
  2. Entity가 화면 종속적으로 변하면, Entity에 화면 처리를 위한 기능들이 추가되며 지저분해 질 수 있다.
  3. 유지보수 측면에서, Entity는 순수하게 유지하는게 좋다. (핵심 비즈니스 로직만 가지도록)
  4. 화면 처리를 위해서는 Form, 혹은 DTO(Data Transfer Object) 를 사용해야 한다.
     DTO : 데이터 전달만을 위한 객체, Getter, Setter만 있음

- MemberController의 문제점

  : List<Member> members = memberService.findMembers(); 에서 Member Entity를 직접 반환한다.

- 이렇게 만들지 말고, 화면에 필요한 데이터만 따로 전송해주는 DTO를 만들어 출력하자!
  특히 API를 만들 땐 절대 Entity를 반환하면 안 된다!

- Why?
  - API는 하나의 스펙이다. 만약 Entity에서 뭔가가 추가된다면, API 명세가 바뀌어버린다.
  - 만약 Entity에 Password같은 보안이 필요한 Field가 있다면, 해당 Field들이 노출될 수 있다.
  - 따라서 API 명세에 맞춰 필요한 내용만을 반납해주는것이 좋다!

## 4. 상품 등록 및 조회

회원 등록/조회와 거의 같다!

- controller/BookForm.class

```java
package jpabook.jpashop.controller;

import lombok.Getter;
import lombok.Setter;

@Getter @Setter
public class BookForm {
    private Long id;

    private String name;
    private int price;
    private int stockQuantity;

    private String author;
    private String isbn;
}
```

- controller/ItemController.class

```java
package jpabook.jpashop.controller;

import jpabook.jpashop.domain.Item.Book;
import jpabook.jpashop.domain.Item.Item;
import jpabook.jpashop.service.ItemService;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;

import java.util.List;

@Controller
@RequiredArgsConstructor
public class ItemController {

    private final ItemService itemService;

    @GetMapping("/items/new")
    public String createForm(Model model){
        model.addAttribute("form", new BookForm());
        return "items/createItemForm";
    }

    @PostMapping("/items/new")
    public String create(BookForm form){
        Book book = new Book();
        book.setName(form.getName());
        book.setPrice(form.getPrice());
        book.setStockQuantity(form.getStockQuantity());
        book.setAuthor(form.getAuthor());
        book.setIsbn(form.getIsbn());

        itemService.saveItem(book);
        return "redirect:/";
    }

    @GetMapping("/items")
    public String list(Model model){
        List<Item> items = itemService.findItems();
        model.addAttribute("items", items);
        return "items/itemList";
    }
}
```

- templates/createItemForm.html

```java
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragments/header :: header" />
<body>
<div class="container">
    <div th:replace="fragments/bodyHeader :: bodyHeader"/>
    <form th:action="@{/items/new}" th:object="${form}" method="post">
        <div class="form-group">
            <label th:for="name">상품명</label>
            <input type="text" th:field="*{name}" class="form-control"
                   placeholder="이름을 입력하세요">
        </div>
        <div class="form-group">
            <label th:for="price">가격</label>
            <input type="number" th:field="*{price}" class="form-control"
                   placeholder="가격을 입력하세요">
        </div>
        <div class="form-group">
            <label th:for="stockQuantity">수량</label>
            <input type="number" th:field="*{stockQuantity}" class="formcontrol"
                   placeholder="수량을 입력하세요">
        </div>
        <div class="form-group">
            <label th:for="author">저자</label>
            <input type="text" th:field="*{author}" class="form-control"
                   placeholder="저자를 입력하세요">
        </div>
        <div class="form-group">
            <label th:for="isbn">ISBN</label>
            <input type="text" th:field="*{isbn}" class="form-control"
                   placeholder="ISBN을 입력하세요">
        </div>
        <button type="submit" class="btn btn-primary">Submit</button>
    </form>
    <br/>
    <div th:replace="fragments/footer :: footer" />
</div> <!-- /container -->
</body>
</html>
```

- templates/itemList.html

```java
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragments/header :: header" />
<body>
<div class="container">
    <div th:replace="fragments/bodyHeader :: bodyHeader"/>
    <div>
        <table class="table table-striped">
            <thead>
            <tr>
                <th>#</th>
                <th>상품명</th>
                <th>가격</th>
                <th>재고수량</th>
                <th></th>
            </tr>
            </thead>
            <tbody>
            <tr th:each="item : ${items}">
                <td th:text="${item.id}"></td>
                <td th:text="${item.name}"></td>
                <td th:text="${item.price}"></td>
                <td th:text="${item.stockQuantity}"></td>
                <td>
                    <a href="#" th:href="@{/items/{id}/edit (id=${item.id})}"
                       class="btn btn-primary" role="button">수정</a>
                </td>
            </tr>
            </tbody>
        </table>
    </div>
    <div th:replace="fragments/footer :: footer"/>
</div> <!-- /container -->
</body>
</html>
```

## 5. 상품 수정

- ItemController에 추가

```java
@GetMapping("items/{itemId}/edit")
    public String updateItemForm(@PathVariable("itemId") Long itemId, Model model){
        Book item = (Book)itemService.findOne(itemId);

        BookForm form = new BookForm();
        form.setId(item.getId());
        form.setName(item.getName());
        form.setPrice(item.getPrice());
        form.setStockQuantity(item.getStockQuantity());
        form.setAuthor(item.getAuthor());
        form.setIsbn(item.getIsbn());

        model.addAttribute("form", form);
        return "items/updateItemForm";
    }

    // 수정을 구현할 때, Id를 다루는데 항상 주의하자.
    // 클라이언트에서 Id를 조작하는 경우도 있는데, 서버단이든 어디든 Validation 필요하다.
    @PostMapping("items/{itemId}/edit")
    public String updateItem(@ModelAttribute("form") BookForm form){
        Book book = new Book();

        book.setIsbn(form.getIsbn());
        book.setPrice(form.getPrice());
        book.setAuthor(form.getAuthor());
        book.setName(form.getName());
        book.setStockQuantity(form.getStockQuantity());
        book.setId(form.getId());

        itemService.saveItem(book);
        return "redirect:/items";
    }
```

- templates/items/updateItemForm.html에 추가

```java
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragments/header :: header" />
<body>
<div class="container">
    <div th:replace="fragments/bodyHeader :: bodyHeader"/>
    <form th:object="${form}" method="post">
        <!-- id -->
        <input type="hidden" th:field="*{id}" />
        <div class="form-group">
            <label th:for="name">상품명</label>
            <input type="text" th:field="*{name}" class="form-control"
                   placeholder="이름을 입력하세요" />
        </div>
        <div class="form-group">
            <label th:for="price">가격</label>
            <input type="number" th:field="*{price}" class="form-control"
                   placeholder="가격을 입력하세요" />
        </div>
        <div class="form-group">
            <label th:for="stockQuantity">수량</label>
            <input type="number" th:field="*{stockQuantity}" class="form-control" placeholder="수량을 입력하세요" />
        </div>
        <div class="form-group">
            <label th:for="author">저자</label>
            <input type="text" th:field="*{author}" class="form-control"
                   placeholder="저자를 입력하세요" />
        </div>
        <div class="form-group">
            <label th:for="isbn">ISBN</label>
            <input type="text" th:field="*{isbn}" class="form-control"
                   placeholder="ISBN을 입력하세요" />
        </div>
        <button type="submit" class="btn btn-primary">Submit</button>
    </form>
    <div th:replace="fragments/footer :: footer" />
</div> <!-- /container -->
</body>
</html>
```

## 6. 변경 감지와 병합 (중요!)

- 준영속 엔티티

  - 영속성 컨텍스트가 더는 관리하지 않는 엔티티를 말한다.
  - 예를 들어 아래와 같이 식별자 (id) 있는 객체의 경우, 객체 자체는 새로운 객체지만 id는 DB가 Generate 하므로, DB와 매칭되는 식별자가 존재한다.

  ```java
  @PostMapping("items/{itemId}/edit")
      public String updateItem(@ModelAttribute("form") BookForm form){
          Book book = new Book();

          book.setIsbn(form.getIsbn());
          book.setPrice(form.getPrice());
          book.setAuthor(form.getAuthor());
          book.setName(form.getName());
          book.setStockQuantity(form.getStockQuantity());
          book.setId(form.getId());

          itemService.saveItem(book);
          return "redirect:/items";
      }
  ```

  - 따라서, 영속성 컨텍스트가 book을 관리하진 않지만, book 개체는 DB의 식별자를 들고 있어서 연관성이 있다.
  - 따라서, 임의로 만들어 낸 객체라도, 식별자를 가지고 있다면 준영속 엔티티라고 볼 수 있다.

- 준영속 엔티티를 수정하는 2가지 방법

  - 변경 감지 (Dirty Check) 기능 사용 (추천)
  - 병합 (Merge) 기능 사용

- 변경 감지 기능 사용 (ItemService.class)

```java
@Transactional
    public void updateItem(Long itemId, Book bookParam){
        //이때 findItem은 영속된 Entity, 따라서 변경점 있다면 JPA가 감지할 수 있다.
        Item findItem = itemRepository.findOne(itemId);

        findItem.setPrice(bookParam.getPrice());
        findItem.setName(bookParam.getName());
        findItem.setStockQuantity(bookParam.getStockQuantity());

        //save 하지 않아도 된다! (findItem이 영속되어 있으므로, Dirty Check 통해 자동으로 저장한다.
    }
```

- 병합 기능 사용

```java
@Transactional
void update(Item item) { //itemParam: 파리미터로 넘어온 준영속 상태의 엔티티
	Item mergeItem = em.merge(item);
}
```

![Untitled](https://github.com/LemonDouble/TIL/blob/main/spring/img/Untitled%2028.png)

- 요약 : 영속 엔티티를 찾아 준영속(우리가 선언한) 엔티티의 값으로 전부 변환한다!

- 병합 (Merge)의 동작 방식

  1. merge()를 실행
  2. 파라미터로 넘어온 준영속 Entity의 식별자 값으로 1차 캐시에서 Entity를 조회
     1. 만약 1차 캐시에 Entity 없다면, DB에서 조회하고 1차 캐시에 저장
  3. 조회한 영속 엔티티(mergeMember)에, 준영속 Entity의 값을 채워 넣는다.
     (준영속 entity의 모든 값을 원래 entity에 밀어 넣는다.)
  4. 영속 상태인 준영속 엔티티를 반환한다.

- 왜 Merge를 사용하는게 안 좋나요?
  - Merge 사용시, 모든 속성이 변경된다. 만약 값이 없다면 null로 업데이트 할 수도 있고, 예상치 못한 값이 업데이트 될 수도 있다.

주의 : 준영속 엔티티가 영속으로 바뀌지 않는다. merge() 함수의 return으로 영속 엔티티를 반환할 뿐, 기존의 준영속 엔티티가 영속 엔티티가 되지 않는다.

- 가장 좋은 해결 방법

  - 엔티티를 변경할 때는 항상 변경 감지를 사용하자!

  - 컨트롤러에서 어설프게 엔티티를 생성하지 말자.
  - 트랜잭션이 있는 서비스 계층에, 식별자(id)와 변경할 데이터를 명확하게 전달하자.
    (파라미터, 또는 DTO)
    itemService.updateItem(itemId, [form.name](http://form.name) , form.price, form.stockQunatity)
  - 트랜잭션에 있는 서비스 계층에서, 영속 상태의 엔티티를 조회하고 엔티티의 데이터를 직접 변경하자.
  - 트랜잭션 커밋 시점에 변경 감지가 실행된다!

## 7. 주문 검색 및 취소

- OrderController.class

```java
package jpabook.jpashop.controller;

import jpabook.jpashop.domain.Item.Item;
import jpabook.jpashop.domain.Member;
import jpabook.jpashop.domain.Order;
import jpabook.jpashop.domain.OrderStatus;
import jpabook.jpashop.repository.OrderSearch;
import jpabook.jpashop.service.ItemService;
import jpabook.jpashop.service.MemberService;
import jpabook.jpashop.service.OrderService;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@Controller
@RequiredArgsConstructor
public class OrderController {

    private final OrderService orderService;
    private final MemberService memberService;
    private final ItemService itemService;

    @GetMapping("/order")
    public String createForm(Model model){

        List<Member> members = memberService.findMembers();
        List<Item> items = itemService.findItems();

        model.addAttribute("members", members);
        model.addAttribute("items",items);

        return "order/orderForm";
    }

    @PostMapping("/order")
    public String order(@RequestParam("memberId")Long memberId,
                        @RequestParam("itemId") Long itemId,
                        @RequestParam("count") int count){

        orderService.order(memberId, itemId, count);
        return "redirect:/orders";
    }

    @GetMapping("/orders")
    public String orderList(@ModelAttribute("orderSearch")OrderSearch orderSearch, Model model){
        List<Order> orders = orderService.findOrders(orderSearch);
        model.addAttribute("orders", orders);

        return "order/orderList";
    }

    @PostMapping("/orders/{orderId}/cancel")
    public String cancelOrder(@PathVariable("orderId") Long orderId){
        orderService.cancelOrder(orderId);
        return "redirect:/orders";
    }
}
```

- templates/order/orderList.html

```java
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragments/header :: header"/>
<body>
<div class="container">
    <div th:replace="fragments/bodyHeader :: bodyHeader"/>
    <div>
        <div>
            <form th:object="${orderSearch}" class="form-inline">
                <div class="form-group mb-2">
                    <input type="text" th:field="*{memberName}" class="formcontrol"
                           placeholder="회원명"/>
                </div>
                <div class="form-group mx-sm-1 mb-2">
                    <select th:field="*{orderStatus}" class="form-control">
                        <option value="">주문상태</option>
                        <option th:each=
                                        "status : ${T(jpabook.jpashop.domain.OrderStatus).values()}"
                                th:value="${status}"
                                th:text="${status}">option
                        </option>
                    </select>
                </div>
                <button type="submit" class="btn btn-primary mb-2">검색</button>
            </form>
        </div>
        <table class="table table-striped">
            <thead>
            <tr>
                <th>#</th>
                <th>회원명</th>
                <th>대표상품 이름</th>
                <th>대표상품 주문가격</th>
                <th>대표상품 주문수량</th>
                <th>상태</th>
                <th>일시</th>
                <th></th>
            </tr>
            </thead>
            <tbody>
            <tr th:each="item : ${orders}">
                <td th:text="${item.id}"></td>
                <td th:text="${item.member.name}"></td>
                <td th:text="${item.orderItems[0].item.name}"></td>
                <td th:text="${item.orderItems[0].orderPrice}"></td>
                <td th:text="${item.orderItems[0].count}"></td>
                <td th:text="${item.status}"></td>
                <td th:text="${item.orderDate}"></td>
                <td>
                    <a th:if="${item.status.name() == 'ORDER'}" href="#"
                       th:href="'javascript:cancel('+${item.id}+')'"
                       class="btn btn-danger">CANCEL</a>
                </td>
            </tr>
            </tbody>
        </table>
    </div>
    <div th:replace="fragments/footer :: footer"/>
</div> <!-- /container -->
</body>
<script>
function cancel(id) {
var form = document.createElement("form");
form.setAttribute("method", "post");
form.setAttribute("action", "/orders/" + id + "/cancel");
document.body.appendChild(form);
form.submit();
}
    </script>
</html>
```
