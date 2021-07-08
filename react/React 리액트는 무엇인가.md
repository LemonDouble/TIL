# React : 리액트는 무엇인가?

분류: React
자료 링크: https://www.inflearn.com/course/react-velopert/dashboard
작성일시: 2021년 7월 7일 오후 4:52

## 1. 프론트엔드 라이브러리란?

- 오늘날의 웹은 복잡한 Application 형태가 많음
- 하지만 Web Application을 직접 구현하려면, 수많은 State를 관리해야 한다
- 프로젝트가 복잡해질수록 DOM 요소 관리, 상태 관리가 복잡해진다.

- Front-End 라이브러리 통해 위와 같은 내용을 무시하고, 핵심 로직 개발에만 집중할 수 있음

## 2. 리액트의 Virtual DOM

- 기존 : MVC, MVVM, MVW 등을 사용 → 공통점으로 Model이 공용으로 사용됨
- View의 값이 변하면, Model이 변하고, Model이 변하면 View가 변함 (양방향 바인딩)
- 이때 Mutation (변화) 는 비싸다. 차라리 Mutation 하지 말고, 그때그때 View를 새로 만들어 버리는 것이 어떨까?
- 하지만 그때그때 DOM 새로 만들고 렌더링 하는 것은 비싸므로, ( [https://velopert.com/3236](https://velopert.com/3236) )   가상의 DOM을 만들고, 기존의 DOM과 비교하여 변화가 필요한 곳만 업데이트 한다.
- Virtual DOM을 통해 매번 DOM 새로 그리고, 바뀐 부분만 업데이트 하는 식으로 작업할 수 있게 된다.

![React%20%E1%84%85%E1%85%B5%E1%84%8B%E1%85%A2%E1%86%A8%E1%84%90%E1%85%B3%E1%84%82%E1%85%B3%E1%86%AB%20%E1%84%86%E1%85%AE%E1%84%8B%E1%85%A5%E1%86%BA%E1%84%8B%E1%85%B5%E1%86%AB%E1%84%80%E1%85%A1%2007d4f0c4dbe6473293c10f7c9a35cdf5/Untitled.png](https://github.com/LemonDouble/TIL/blob/main/react/img/Untitled.png)

- 참고자료 : React And the Virtual DOM (한글자막 있음 : [https://www.youtube.com/watch?v=muc2ZF0QIO4](https://www.youtube.com/watch?v=muc2ZF0QIO4) )

## 3. React를 특별하게 만드는 점

- Vue, Marko, Maquette, Mithril... 등, 다른 라이브러리도 Virtual DOM 사용한다.
- 그럼 React만의 장점만은 무엇일까? (주관)
    1. 생태계가 크고 뜨거운 편이다. 
    2. 사용하는 곳이 많다. (Airbnb, BBC, Cloudflare, Twitch, Facebook..)
    3. 등등..
