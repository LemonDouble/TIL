# 함수형 프로그래밍과 JavaScript ES6+ - ES6에서의 순회와 이터러블:이터레이터 프로토콜

분류: Javascript
작성일시: 2021년 9월 15일 오후 9:24

## 1. 기존과 달라진 ES6에서의 리스트 순회

- 기존(ES5 이하) 에서의 리스트 순회

```jsx
const list = [1,2,3];
for (var i =0; i < list.length; i++){
    console.log(list[i]);
}
//1, 2, 3
```

- ES6 이상에서의 리스트 순회

```jsx
const list = [1,2,3];
for (const a of list){
    console.log(a);
}
const str = 'abc'; //string도 가능
for (const a of str){
    console.log(a);
}
```

## 2. Array, Set, Map을 통해 알아보는 이터러블/이터레이터 프로토콜

```jsx
const arr = [1,2,3];
const set = new Set([1,2,3]);
const map = new Map([['a',1], ['b', 2], ['c',3]]);

// 전부 아래와 같이 접근 가능하다.
for (const a of arr) console.log(a);
for (const a of set) console.log(a);
for (const a of map) console.log(a);

//하지만, 위의 ES5 이하에서의 순회처럼 단순히 for문 돌면서 조회하는 방식은 아니다.
arr[0]; arr[1]; arr[2]; //로 접근은 가능하지만
set[0]; set[1]; set[2];
map[0]; map[1]; map[2]; //set, map은 위처럼 접근이 불가능하다.
// 즉,for of문은 우리가 아는 기존 for문처럼 동작하지 않는다!
```

- Symbol.iterator 라는 Symbol이 있다.

```jsx
const arr = [1,2,3];
arr[Symbol.iterator]; // ƒ values() { [native code] } , 어떤 function이 있다.
arr[Symbol.iterator] = null; // 강제로 해당 function을 null로 비운다.
for(const a of arr) console.log(a); // Uncaught TypeError: arr is not iterable

// Set, Map 동일하다!
```

- 이터러블 / 이터레이터 프로토콜
    - 이터러블 : 이터레이터를 리턴하는 [Symbol.iterator]() 를 가진 값.
    - 이터레이터 : { value, done } 객체를 리턴하는 next()를 가진 값.
    - 이터러블 / 이터레이터 프로토콜 : 이터러블을 for..of, 전개 연산자 등과 함께 동작하도록 한 규약

```jsx
const arr = [1,2,3];
arr[Symbol.iterator]; // ƒ values() { [native code] }
const iter = arr[Symbol.iterator](); // Array Iterator {}
iter.next(); // {value: 1, done: false}
iter.next(); // {value: 2, done: false}
iter.next(); // {value: 3, done: false}
iter.next(); // {value: undefined, done: true}
```

- for of문 : value 값을 출력하다, done이 true가 되면 for문에서 빠져나옴.
    - Set, Map 또한 마찬가지로, Iterator Protocol 따르고 있기 때문에 순회가 가능하다.

- map은 keys, values, entries 함수를 제공하는데, 각각의 iterator를 리턴한다.

```jsx
//value도 동일!
const map = new Map([['a',1], ['b', 2], ['c',3]]);
const keyIter = map.keys();
keyIter.next(); //{value: 'a', done: false}
keyIter.next(); //{value: 'b', done: false}
keyIter.next(); //{value: 'c', done: false}
keyIter.next(); //{value: undefined, done: true}

//entries는 둘 다 반환한다.
const entryIter = map.entries();
entryIter.next(); //{value: Array(2) //['a', 1] , done: false}
```

- 이때, 재미있는 점이 있다.

```jsx
const map = new Map([['a',1], ['b', 2], ['c',3]]);
const keyIter = map.keys();
const newIter = keyIter[Symbol.iterator](); //key Iterator도 Symbol.iterator가 있다.
//단, key Iterator의 Symbol.iterator() 호출하면 자기 자신을 반환한다. (value 등도 동일)

newIter.next(); //{value: 'a', done: false}
keyIter.next(); //{value: 'b', done: false}
newIter.next(); //{value: 'c', done: false}
keyIter.next(); //{value: undefined, done: true}
```

## 3. 사용자 정의 이터러블, 이터러블/이터레이터 프로토콜 정의

- (2) 에서, for of와 어떻게 이터러블 프로토콜이 작동하는지 보았다.

- 사용자 정의 이터레이터를 만들며, 어떻게 이터레이터가 작동하는지 알아보자.

```jsx
// 3, 2, 1 반환하고 끝나는 iterable
const iterable = {
	//iterable은 Symbol.iterator가 존재한다.
  [Symbol.iterator]() {
    let i = 3;
    return {
      next() {
				// i가 0이 되기 전까진 하나씩 감소한 값을, 0이 되면 done:true를 반환한다.
        return i == 0 ? { done: true } : { value: i--, done: false };
      },
    };
  },
};

let iterator = iterable[Symbol.iterator]();
iterator.next(); //{value: 3, done: false}
iterator.next(); //{value: 2, done: false}

for(const a of iterator) console.log(a); // 3, 2, 1 
```

- 하지만, 우리가 만든 iterator는 well-formed iterator 아니다.
    - array의 iterator를 살펴보자.

```jsx
const arr = [1,2,3];
const arrIter = arr[Symbol.iterator]();
arrIter.next(); // {value: 1, done: false}
// 멈췄던 지점부터 다시 시작할 수 있다
for(const a of arrIter) console.log(a); // 2,3
```

- 하지만 우리의 iterator는 불가능하다.

```jsx
let iterator = iterable[Symbol.iterator]();
iterator.next(); //{value: 3, done: false}
for (const a of iterator) console.log(a); //Uncaught TypeError: iterator is not iterable
```

- 따라서, Well-formed iterator로의 변환이 필요하다.

```jsx
const iterable = {
  [Symbol.iterator]() {
    let i = 3;
    return {
      next() {
        return i == 0 ? { done: true } : { value: i--, done: false };
      },
			// 자기 자신을 반환하도록 수정!
			[Symbol.iterator]() { return this;} 
    };
  },
};
```

- 이제 iterator가, 우리가 생각하는 것 처럼 작동한다!

```jsx
let iterator = iterable[Symbol.iterator]();
iterator.next(); //{value: 3, done: false}
for (const a of iterator) console.log(a); //2, 1
```

- 이 이터러블/이터레이터 프로토콜은 JS 내장 객체들만 지원하는 것이 아니다.
    - 많이 사용하는 오픈소스 라이브러리 등은 대부분 이터러블/이터레이터 프로토콜을 지원!
        - Immutable.js..
    - 브라우저에서 사용 가능한, Web API(?) 등도 이터러블/이터레이터 프로토콜을 따른다.
        - for ( const a of document.querySelectAll('*')) log (a);

## 4. 전개 연산자

- 전개 연산자도 마찬가지로, Iterator, Iterable 프로토콜을 따른다.

```jsx
const arr = [1,2];
[...arr, ...[3,4]] // [1, 2, 3, 4]

arr[Symbol.iterator] = null;
// 전개 연산자도 Iterable Protocol 사용하는거 알 수 있다! 
[...arr, ...[3,4]] // Uncaught TypeError: arr is not iterable
```