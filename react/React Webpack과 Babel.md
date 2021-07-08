# React : Webpack과 Babel

분류: React
작성일시: 2021년 7월 7일 오후 5:30

## 1. Webpack이란 무엇인가?

- 웹 어플리케이션을 구현하는 몇십, 몇백개의 자원들을 하나의 파일로 병합 및 압축해주는 도구
- 이러한 작업을 모듈 번들링(Module Bundling) 이라고 함.
- 이러한 방법을 통해 네트워크 접속 부하를 줄여 더 빠른 서비스를 제공하고, Module 단위, File 단위 관리를 지원하여 파일 관리를 편하게 한다

![React%20Webpack%E1%84%80%E1%85%AA%20Babel%20e685bba1e95d4651aebc853f01f40479/Untitled.png](https://github.com/LemonDouble/TIL/blob/main/react/img/Untitled.png)

- 참고자료 : [Webpack] 웹팩 개념 알아보기 ( [https://velog.io/@mnz/Webpack-웹팩-개념-알아보기](https://velog.io/@mnz/Webpack-%EC%9B%B9%ED%8C%A9-%EA%B0%9C%EB%85%90-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0) )
- 참고자료 : Webpack 이란 무엇인가 ( [https://webclub.tistory.com/635](https://webclub.tistory.com/635) )

## 2. Babel이란 무엇인가?

- Javascript 변환 도구 ( [https://babeljs.io/](https://babeljs.io/) )
- 구형 브라우저 등은 최신 JS 문법을 지원하지 않음.
- 따라서 구형 브라우저가 이해할 수 있도록 최신 코드를 변환해주는 도구
- JSX 라는 확장된 JS 문법을 사용하기 위해서도 필요
