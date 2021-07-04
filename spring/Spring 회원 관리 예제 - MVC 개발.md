# Spring : 회원 관리 예제 - MVC 개발

분류: Spring
작성일시: 2021년 7월 4일 오후 6:15

## 1. HomeController (Home 화면)

- main/java/project명/controller/Homecontroller (Class)

```java
package rabbitprotocol.fileuploadrabbitprotocolbackend.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class HomeController {

    @GetMapping("/")
    public String home(){
        return "home";
    }
}
```

- main/resources/templates/home.html (html)

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<body>
<div class="container">
    <div>
        <h1>Hello Spring</h1>
        <p>회원 기능</p>
        <p>
            <a href="/members/new">회원 가입</a>
            <a href="/members">회원 목록</a>
        </p>
    </div>
</div> <!-- /container -->
</body>
</html>
```

- TIP :  Spring은 먼저 "/"로 route되는 Controller 있는지 먼저 찾아보고, 만약 Controller 없다면 static/index.html을 찾아본다. 따라서, 지금은 static에 있는 index.html은 무시된다 (후순위)

## 2. MemberController - 등록 (회원등록)

- main/java/project명/controller/MemberController (수정)

```java
package rabbitprotocol.fileuploadrabbitprotocolbackend.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import rabbitprotocol.fileuploadrabbitprotocolbackend.domain.Member;
import rabbitprotocol.fileuploadrabbitprotocolbackend.service.MemberService;

@Controller
public class MemberController {
    private final MemberService memberService;

    @Autowired
    public MemberController(MemberService memberService) {
        this.memberService = memberService;
    }

    @GetMapping("/members/new")
    public String createForm(){
        return "members/createMemberForm";
    }

    @PostMapping("/members/new")
    public String create(MemberForm form){
        Member member = new Member();
        member.setName(form.getName());

        memberService.join(member);

        //회원가입이 끝나면 Home 화면으로 redirect
        return "redirect:/";
    }

}
```

- main/java/project명/controller/MemberForm (Class)

```java
package rabbitprotocol.fileuploadrabbitprotocolbackend.controller;

public class MemberForm {
    private String name;
    // <input type="text" id="name" name="name" placeholder="이름을 입력하세요"> 의 name과 매칭되어 값이 들어온다.

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

- resources/templates/members/createMemberForm.html

```java
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<body>
<div class="container">
  <form action="/members/new" method="post">
    <div class="form-group">
      <label for="name">이름</label>
      <input type="text" id="name" name="name" placeholder="이름을 입력하세요">
    </div>
    <button type="submit">등록</button>
  </form>
</div> <!-- /container -->
</body>
</html>
```

- <input type="text" id="name" name="name" placeholder="이름을 입력하세요"> 에서, name= "name" 부분이 key가 된다.
- MemberForm의 name과, input type ~~ name="name" 부분이 매칭되어 값이 설정된다. (setter, getter 필요)

## 3. MemberController - 조회 (회원조회)

- memberController에 추가

```java
@GetMapping("/members")
    public String list(Model model){
        List<Member> members = memberService.findMembers();
        model.addAttribute("members", members);

        return "members/memberList";
    }
```

- resources/templates/members/memberList.html

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<body>
<div class="container">
  <div>
    <table>
      <thead>
      <tr>
        <th>#</th>
        <th>이름</th>
      </tr>
      </thead>
      <tbody>
      <tr th:each="member : ${members}">
        <td th:text="${member.id}"></td>
        <td th:text="${member.name}"></td>
      </tr>
      </tbody>
    </table>
  </div>
</div> <!-- /container -->
</body>
</html>
```

이때,  <td th:text="${member.id}"></td>은 thymeleaf 문법.

id/name은 private인데, getter을 통해 접근한다.