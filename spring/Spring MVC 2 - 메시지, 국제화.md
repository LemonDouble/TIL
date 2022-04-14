# Spring MVC 2 - 메시지, 국제화

## 1. 소개

- 만약, 통일된 경험을 제공하기 위해 특정 단어를 다른 단어로 대체하길 원한다면 어떨까?
    - 예시 : 대출 → 돈 빌리기
    - HTML에 하드코딩 해 놨다면, 직접 Ctrl + F 한 후 찾아야 한다.
    - 이 과정에서 Human Error도 생길 가능성이 높다

- 따라서, 하나의 Message Source를 관리할 수 있다면, 국제화 및 메세지 관리가 편해진다.
    - resources 폴더에 [messages.properties](http://messages.properties) 라는 메세지 관리용 파일을 만들고
    
    ```java
    hello=안녕
    #{0} -> Parameters
    hello.name=안녕 {0} {1}
    
    label.item=상품
    label.item.id=상품 ID
    label.item.itemName=상품명
    label.item.price=가격
    label.item.quantity=수량
    page.items=상품 목록
    page.item=상품 상세
    page.addItem=상품 등록
    page.updateItem=상품 수정
    button.save=저장
    button.cancel=취소
    ```
    
    - 위와 같이 저장해 놓은 뒤, HTML에서는
    
    ```java
    <label for="itemName" th:text="#{item.itemName}"></label>
    ```
    
    - 와 같은 방법으로 불러와서 사용할 수 있다.

- 또한, messages_en.properties 와 같이 국제화 지원도 간편하게 할 수 있다.
    - 어떤 국가에서 접근했는지는 HTTP Request 협상 Header 를 통해 알 수 있다.

### 2. Spring Message Source 설정

- Spring boot 사용시 자동으로 MessageSource 빈을 등록해 줌
- 일반 Spring 사용한다면, 직접 설정 파일 만들어서 Bean 등록해줘야 한다.

### 나머지는 필요할 때 찾아보자 여기까지만..