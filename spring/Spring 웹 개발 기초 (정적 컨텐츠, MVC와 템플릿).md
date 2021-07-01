# Spring : 웹 개발 기초 (정적 컨텐츠, MVC와 템플릿 엔진, API)

분류: Spring
작성일시: 2021년 6월 29일 오후 9:34

# 웹 개발의 세 가지 방법

1. 정적 컨텐츠
2. MVC와 템플릿 엔진
3. API : Android / Ios, 혹은 Vue, React등 사용할 때, 서버끼리 통신할 때

## 1. 정적 컨텐츠 : Spring boot는 정적 컨텐츠를 기본적으로 제공

- resources → static 폴더
- [https://docs.spring.io/spring-boot/docs/2.3.1.RELEASE/reference/html/spring-boot-features.html#boot-features-spring-mvc-static-content](https://docs.spring.io/spring-boot/docs/2.3.1.RELEASE/reference/html/spring-boot-features.html#boot-features-spring-mvc-static-content)

## 2. MVC : Model, View, Controller

- JSP : View에 모든 로직이 들어가 있었다.
- View : Render에만 집중, Cotnroller랑 Model에서 비즈니스 모델 처리 후, View는 출력만 담당

```java
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;

@Controller
public class HelloController {

    @GetMapping("hello-mvc")
    public String helloMVC(@RequestParam("name") String name, Model model){
        model.addAttribute("name",name);
        return "hello-template";
    }
}
```

Controller (java→project명→controller 폴더)

- View Resolver가 hello-template.html을 찾아 render

```html
<html xmlns:th="http://www.thymeleaf.org">
<body>
<p th:text="'hello ' + ${name}">hello! empty</p>
</body>
</html>
```

View (resources→templates 폴더)

## 3. API

```java
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class HelloController {
    @GetMapping("hello-api")
    @ResponseBody //이 때 Body는 HTML의 Body가 아니라, HTTP 프로토콜의 Body
    public Hello helloApi(@RequestParam("name") String name){
        Hello hello = new Hello();
        hello.setName(name);

        return hello; //Object 리턴시 Json으로 return
    }

    static class Hello{
        private String name;

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }
    }
}
```

Controller (java→project명→controller 폴더)

Object 반환시 Json 형태로 반환한다.

[localhost](http://localhost):8080/hello-api?name=Spring! 실행시

```json
{"name":"Spring!"}
```

- @ResponseBody 사용시 viewResolver 사용하지 않음. 대신 HttpMessageConverter 동작
- 기본 문자처리 : StringHttpMessageConverter
- 기본 객체처리 : MappingJackson2HttpMessageConverter

- 클라이언트가 HTML 메세지 보낼 때, HTTP 프로토콜에 Accept 헤더 있는데, 이 값에 맞춰 적절한 HttpMessageConverter 선택된다. (만약 XML 요구하면 XML 줄 수 있음)

## TIP!

- IntelliJ getter / setter 자동완성 : Alt + Insert (Windows)
- 자동완성 : Ctrl + Shift + Space