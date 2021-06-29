# Spring : Welcome page와 Build

분류: Spring
자료 링크: https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EC%9E%85%EB%AC%B8-%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8/lecture/49573
작성일시: 2021년 6월 29일 오후 3:51

## Welcome Page 만들기 (static)

- src → resources → static에 index.html 생성
- index.html이 Weclome page 기능 수행

## Welcome Page 만들기 (Data 전달)

- src→main→java→프로젝트명→ controller 폴더 생성
- controller 폴더 안에 HelloController라는 클래스 만들고 다음과 같이 코드 복사

```java
@Controller
public class HelloController {

	@GetMapping("hello")
	public String hello(Model model) {
		model.addAttribute("data", "hello!!");
		return "hello"; //hello.html을 render한다.
	}

}
```

"data" 이름으로 "hello" 라는 값이 전달.

- src→templates에 hello.html 파일 추가

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Hello</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>
<p th:text="'안녕하세요. ' + ${data}" >안녕하세요. 손님</p>
</body>
</html>
```

위에 "data" 가 hello!! 이므로, "안녕하세요. hello!!" 출력

## Spring 공식 Document

[https://spring.io/](https://spring.io/) 들어가서 project → spring boot → 2.3.1 → document 검색

## Thymeleaf 템플릿 엔진 Document

[thymeleaf.org](http://thymeleaf.org)

## 빌드 방법

1. ./gradlew build
2. d build/libs
3. java -jar hello-spring-0.0.1-SNAPSHOT.jar