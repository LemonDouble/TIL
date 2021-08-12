# JavaScript 기초

분류: Javascript
자료 링크: https://youtu.be/KF6t61yuPCY
작성일시: 2021년 8월 10일 오전 10:16

- 편의상 다른 언어와 비슷한 부분은 생략함. (if, for 등..)
- JS : 인터프리터 언어

## 1. 변수

- let : Block Scope
    - 변수 재선언 불가능
    - 변수 재할당은 가능
- const
    - 변수 재선언 불가능
    - 변수 재할당 불가능
    - (관례) 대문자를 사용하는 것이 좋다. (cosnt MAX_SIZE = 99;)

## 2. 자료형

- String
    - 백틱 (1 옆에 `) : 문자열 내부 변수를 표현할 때 사용

    ```java
    const name = "mike"
    const msg = `My name is ${name}!`; // My name is mike!
    const msg2 = `i am ${30+2} years old!`; // i am 32 years old!
    const msg3 = `i am ${'30'+2} years old!`; // i am 302 years old!
    ```

- Number
    - +,-,*,/,%
    - 1 / 0 = Infinity
    - "Mike"/2 = NaN (Not a Number)
- Boolean
    - true, false
- null / undefined
    - null: 값이 없음 (주의 : typeof 함수로 검색 시 object 나옴)
    - undefined : 할당되지 않음

    ```java
    let user = null; // null
    let user;  // undefined
    ```

## 3. alert, prompt, confirm

- prompt
    - 확인/취소창이 있는 input 창을 띄운다.
    - 취소시 null 반환한다.
    - default값 세팅 가능하다.

    ```java
    const name = prompt("예약일을 입력해주세요.", "2020-10-")
    //prompt 창에서 자동으로 2020-10까지 입력되어 있다!
    ```

- confirm
    - 확인/취소 창이 있는 확인 창을 띄운다.
    - true(확인), false(취소) 반환한다.

- 단점
    - 스크립트가 확인을 누르기 전까지 일시 정지된다.
    - 디자인 불가

## 4. 형변환

- String(), Number(), Boolean()
- Number(true) → 1 , Number(false) → 0
- Boolean
    - false : 0, "", null, undefined, NaN
    - 그 외 전부 true 반환

## 5. 연산자

- 거듭제곱 : **

    ```jsx
    2 ** 2 // 4
    2 ** 3 // 8
    2 ** 4 // 16
    ```

- 비교 연산자
    - == : 변환 전 형변환을 수행 ( 0 == "0" // true)
        - 하지만 NaN, +0, -0은 따로 관리
        - NaN == NaN // false
        - +0 == -0 //true
    - === : type도 고려하여 비교

## 6. 논리 연산자

- &&(AND), ||(OR), ! (NOT)
- Short-circuit evaluation 적용
    - 만약 a() = true, b() = true, c = false() 인 경우, if(a() || b() || c()) 시 a()만 호출하고 b,c는 호출되지 않음
    - 만약 if(c() && b () && a()) 라면 c()만 호출됨

- OR 연산자의 첫 번째 truthy 탐색

    ```jsx
    let firstName = "";
    let secondName = "";
    let thirdName = "트루먼";

    alert(fistName || secondName || thirdName || "익명") // 첫 번째 truthy인 트루먼 반환 
    ```

- 같은 방식으로, AND 연산자는 첫 번째 Falsy를 반환

## 7. 함수

```jsx
function sayHello(name){
	console.log(`${name}, hello!`);
}
```

- 만약 return문이 없다면 undefined 를 반환한다. (위 sayHello는 undefined return한다.)

## 8. 전역 변수와 지역 변수

```jsx
let name = "Mike"

function sayHello(name){
	console.log(name);
}

sayHello(); //undefined
sayHello('Jane'); // Jane
```

- 매개변수가 없는 sayHello();에서, parameter가 없는 경우 undefined가 sayHello() function 내의 지역변수로 복사되므로 전역 변수를 덮고 undefined가 출력된다.
- 가급적 지역 변수를 사용하도록 하자.

## 9. 함수 표현식

- 함수 선언문

    ```jsx
    //어디에서나 호출 가능하다!
    sayHello();

    function sayHello(){
    	console.log(`hello!`);
    }
    ```

    - 초기화 단계에서, 모든 함수의 선언부를 찾아 해당 Scope를 가장 위로 옮긴다.
    - 이를 호이스팅(Hoisting) 이라고 한다.
    - 함수 선언문 외에도 var에도 같은 역할 한다.

- 함수 표현식

    ```jsx
    //여기서 호출 불가능하다!

    let sayHello = function(){
    	console.log('Hello');
    }

    sayHello();
    ```

    - 인터프리터가 함수 표현식을 만나야 비로소 사용이 가능해진다.

- 화살표 함수

    ```jsx
    //기존의 함수 표현식
    let add = function(num1, num2){
    	return num1 + num2;
    }

    //function을 Arrow로 바꿈
    let add = (num1, num2) => {
    	return num1 + num2;
    }

    //return문은 소괄호로 바꿀 수 있음
    let add = (num1, num2) => (
    	num1 + num2;
    )

    //return문이 1줄이라면 괄호도 생략할 수 있음
    let add = (num1, num2) => num1 + num2;

    //만약, 파라미터가 하나라면 파라미터의 괄호도 생략할 수 있음
    let sayHello = name => `Hello, ${name}`;
    ```

## 10. 객체 (Object)

- 프로퍼티 : Object 내 구성요소

```jsx
//선언 
const superman =
	name : 'clark',
	age : 33,
}

//접근
superman.name //'clark'
superman['age'] // 33

//추가
superman.gender = 'male';
superman['hairColor'] = 'black';

//삭제
delete superman.hairColor;
```

- 단축 프로퍼티 활용

```jsx
const name = 'clark';
const age =33;

const superman = {
	name : name, // name : 'clark'
	age : age // age : 33
	gender : 'male',
}

//아래와 같이 선언해도 위와 동일하다!

const superman = {
	name,
	age,
	gender : 'male',
}
```

- 프로퍼티 존재 여부 확인

```jsx
const superman =
	name : 'clark',
	age : 33,
}

superman.birthDay; //undefined

'birthDay' in superman //false
'age' in superman //true
```

## 11. method와 this

- 일반적인 함수 메소드 추가

```jsx
const superman =
	name : 'clark',
	age : 33,
	fly : function(){
		console.log("fly!");
	}
}

//생략하여 더 간단히 작성 가능
const superman =
	name : 'clark',
	age : 33,
	fly(){
		console.log("fly!");
	}
}
```

- this (일반 함수 선언에선 예측한대로 작동하지만, Arrow function에선 다르게 작용한다)

```jsx
const mike = {
	name : 'Mike',
	sayHello : function(){
		console.log(`Hello, I'm ${this.name}`);
	}
}

mike.sayHello(); // Hello, I'm Mike

const mike = {
	name : 'Mike',
	sayHello : ()=>{
		console.log(this);
	}
}

mike.sayHello(); // Window {0: global, window: Window, self: Window, document: document, name: "", location: Location, …}
```

- Arrow Function에서는 this가 browser에서는 window 전역객체, nodejs에서는 global 전역객체가 호출된다.

## 12. 배열

- 내장함수
    - length : 배열의 길이 반환
    - push() : 배열 끝에 추가
    - pop() : 배열 끝 요소 제거
    - shift, unshift : 배열 가장 앞에 제거/추가