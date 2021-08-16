# Javascript 중급

분류: Javascript
자료 링크: https://youtu.be/4_WLS9Lj6n4
작성일시: 2021년 8월 10일 오후 4:53

## 1. 변수의 생성, var,let,const, TDZ, Hoisting

- 변수의 생성 과정
    1. 선언 단계
    2. 초기화 단계 : undefined를 할당해주는 단계
    3. 할당 단계 : 값을 할당해주는 단계 ( name = 'Mike')

- var :  선언된 변수를 재선언 가능, 선언 전에 사용 가능 (Hoisting에 의해), TDZ의 영향 받지 않음
    - var은 선언 및 초기화 단계를 한번에 진행
    - function scope

- let : 선언된 변수를 재선언 불가, 선언 전 사용 불가, TDZ의 영향 받음
    - let도 hoisting 되지만 TDZ의 영향을 받아 ReferenceError가 난다.
    - let은 선언 / 초기화 단계가 분리,
    - block scope

- const : 선언+ 초기화 + 할당이 동시에 진행
    - block scope

- TDZ : Temporal Dead Zone : 할당 전 사용 불가!

```jsx
//var의 사용
console.log(name); //undefined

var name = 'Mike';

--------------------------
//var의 hoisting 이후
var name; //선언 및 undefined 초기화를 진행
console.log(name);  //undefined 출력
name = 'Mike';

--------------------------
//let의 사용
console.log(name); //undefined

let name = 'Mike';

--------------------------
//let의 hoisting 이후 (let도 hoisting됨)

let name; //이때 선언은 되지만, 초기화는 안 됨
console.log(name); //따라서 여기서 referenceError 발생
//여기까지가 TDZ
name = 'Mike'
```

## 2. 생성자 함수

- 객체 리터럴

```jsx
//이런거
let user = {
	name : 'Mike',
	age : 30,
}
```

- 생성자 함수

```jsx
//첫 글자는 대문자로 (일반 함수와 구분 위한 관례)
function User(name, age){
	this.name = name;
	this.age = age;
}

let user1 = new User('Mike', 30);
let user2 = new User('Jane', 20);
```

## 3. 객체 메소드, 계산된 프로퍼티 (Object Method, Computed property)

```jsx
const user = {
	name : 'Mike',
	age : 30,
}
```

- Object 메소드
    - Object.assign(): 객체 복제 (deep copy)

        ```jsx
        // 객체 복사, 첫 번째 프로퍼티는 초기값
        const newUser = Object.assign({} , user);

        //없는 프로퍼티를 주면 user + 없는 프로퍼티가 된다.
        //즉, name, age, gender 세개 가진다.
        const newUser = Object.assign({gender:'male'} , user);

        //있는 프로퍼티를 주면 user의 해당 프로퍼티가 덮어쓰기 된다.
        const newUser = Object.assign({name:'Tom'} , user);

        //여러 객체를 합칠 수도 있다.
        //user-> name / info1 -> age / info2 -> gender만 있는 경우
        const newUser = Object.assign(user, info1, info2);

        //user에 info1, info2 순서대로 합쳐진다.
        ```

    - Object.keys(user) : key 배열 반환 ( ["name", "age"] )
    - Object.values(user) : value 배열 반환 ( ["Mike", 30] )
    - Object.entries(user) : 키/값 배열 반환 ( [ ["name", "Mike"] , ["age", 30] ] )

        배열 속 배열을 반납

- 계산된 프로퍼티 (Computed property)

    ```jsx
    let n = "name"
    let a = "age";

    const user= {
    	//Computed property
    	[n] : "Mike",
    	[a] : 30,
    	[1+4] : 5,
    };

    //위와 아래 객체는 동일
    const user= {
    	name : "Mike",
    	age : 30,
    	5 : 5,
    };

    //아래와 같이, Key가 고정되지 않은 객체를 만들 때도 사용할 수 있다.
    function makeObj(key, val){
    	return {
    		[key] : val,
    	}
    }

    const obj = makeObj("성별", "male");
    ```

## 4. 심볼 (Symbol)

- 만약 서드파티 (외부 라이브러리) 등에서 Object를 가져와서 사용하는데, 새로운 식별자를 추가하고 싶다면?
    - 새 프로퍼티를 만들 수도 있지만, 다른 사람이 만든 Object의 프로퍼티와 중복되어 값을 덮어쓰거나 할 수도 있다.
    - Symbol을 사용하여 key를 만들면, 해당 Key가 중복되지 않음이 보장된다.
    - 숨김(hidden) 프로퍼티를 만들고 싶을 때 사용할 수도 있으나, 아래와 같이 완벽하게 은닉은 할 수 없다. (Object.getOwnPropertySymbols())

- 유일한 식별자를 만들 때 사용

```jsx
const a = Symbol(); //new 사용하지 않음
const b = Symbol();

a === b // false
a == b // false

//문자열은 Symbol 생성에 영향을 끼치지 않음.
//주석과 비슷한 역할
const id = Symbol('id');

const user = {
	name : 'Mike',
	age : 30,
	[id] : 'myid',
}

id.description; // id, Symbol의 프로퍼티 보기, 

//이때 Symbol은 Object.keys, values, entries등에 의해 반환되지 않음
//즉, name, age만 반환된다.

//심볼만 따로 보고싶다면
Object.getOwnPropertySymbols(user); // [Symbol(id)]

//심볼과 키를 전부 보고싶다면
Reflect.ownKeys(user); //["name", "age", Symbol(id)]
```

- 전역 심볼

```jsx
//하나의 심볼을 보장해 줌.
//없으면 만들고, 있으면 해당 Symbol을 가져옴
//같은 Symbol을 공유하는 방법
//Symbol.for()로 사용

const id1 = Symbol.for('id');
const id2 = Symbol.for('id');

id1 === id2 //true
Symbol.keyFor(id1) // id, 전역 심볼에서만 사용 가능
```

## 5. 숫자, 수학 관련 method

- toString() : 10진수 → 2진수/16진수

```jsx
let num = 10;
num.toString(); //"10"
num.toString(2); // "1010"

let num2 = 255;
num2.toString(16); // "ff"
```

- Math.PI; : 3.1425....
- Math.ceil() : 올림, floor() : 내림, round() : 반올림
    - 활용 : 소숫점 둘째자리까지 표현 (셋째 자리에서 반올림)
        - let userRate = 30.1234;
        1. *100 후 Math.round() 후 /100
        2. userRate.toFixed(2); // "30.12"
            - 이 때, toFixed는 String 반환하므로 Number()로 바꿔서 사용해 준다.
- isNaN() : 해당 숫자가 NaN임인지 확인
    - NaN == NaN //false 이므로, 숫자가 NaN인지 확인하는 유일한 방법
- parseInt() : String을 숫자로, 왼쪽부터 읽을 수 있는 부분까지는 읽는다.

    ```jsx
    let a = '10px';
    let b = parseInt(a);

    b //10

    let c = 'px10';
    let d = parseInt(c);

    d // NaN

    //Parameter 통해서 진수변환도 가능하다.
    let redColor = 'f3'

    parseInt(redColor) // NaN
    parseInt(redColor, 16) // 243 (16진수를 10진수로)
    parseInt('11',2) // 3 (2진수를 10진수로)
    ```

- ParseFloat() : ParseInt()와 동일하지만, 부동소수점을 반환한다.
- Math.random() : 0~1 사이의 무작위 숫자 생성
- Math.pow(2,10) : 거듭제곱을 구해줌 (1024)
- Math.sqrt(16) : 제곱근을 구해줌

## 6. 문자열 메소드 (String Methods)

- 큰 따옴표, 작은 따옴표는 구분이 없다.
- 백틱(`) 은 여러 줄을 표현할 때 \n 없이 작성할 수 있다.

- str.function()..
    - length : 문자열의 길이 반환
    - a[2] 와 같이 문자에 접근 가능, 하지만 한 글자만 바꿀 순 없다.
    - toUpperCase(), toLowerCase() : 모든 글자를 대문자로, 소문자로
    - indexOf('찾는 문자') : 찾는 문자가 있으면 해당 index를, 없으면 -1을 반환
    - slice(n,m) : index n부터 m까지의 문자열을 반환
        - 이때, m이 없으면 문자열 끝까지, 양수면 그 숫자까지 (m은 포함 x) , 음수면 끝에서부터 셈
        - lest desc = "abcdefg"
        - desc.slice(2) // "cdefg"
        - desc.slice(0,5) // "abcde"
        - desc.slice(2,-2) // "cde"
    - substring(n,m) : n과 m 사이의 문자열을 반환, slice와 유사하지만 n,m이 바뀌어도 동작, 음수는 0으로 인식
    - substr(n,m) : n부터 시작해서 m개를 가져옴
    - trim() : 앞뒤 공백 제거

## 7. 배열 메소드 (Array Methods)

- arr.function()..
    - splice(n,m) : 특정 요소 지움, n번째부터 m개까지, 리턴값은 삭제된 요소들
    - slice(n,m) : n부터 m까지 반환
    - concat(arr2,arr3,...) : 현 array에 arr2, arr3.. 이어서 반환한다.
    - forEach ((item, index, arr) ⇒ { } );
    - indexOf, lastIndexOf : 발견한 해당 요소의 index 반환, 없으면 -1,
        - indexOf(3,3) : 두 번째 파라미터는 시작 위치
    - includes() : 포함하는지 확인 (true/false)
    - find(function), findIndex(function) : 첫 번째 true값만 반환, 없으면 undefined 반환

        ```jsx
        let arr = [1,2,3,4,5];

        //첫 번째 짝수 찾기
        const result = arr.find((item) => {
            return item %2 === 0;
        });

        console.log(result); //2, 첫 번째 true값만 반환된다!

        //아래와 같이 element, index, arr가 파라미터로 전달된다.
        const result = arr.find((element,index, arr) => {
            console.log(element);
            console.log(index);
            console.log(arr);
            return element % 2 === 0;
        });

        //Object를 찾는 예제

        let userList = [
        	{name : "A", age : 30},
        	{name : "B", age : 27},
        	{name : "C", age : 10}
        ];

        //미성년자 찾기
        const result = userList.find((user) =>{
        	if(user.age < 19){
        		return true;
        	}else{
        		return false;
        	}
        });

        console.log(result); // {name: "C", age:10}
        ```

    - filter(function) : find와 사용법 동일, 하지만 만족하는 모든 값을 배열로 반환
    - map(function) : 각 item에 대해 함수 적용하고 새로운 배열을 반환

        ```jsx
        let arr = [1,2,3,4,5];
        let result = arr.map((item) => {return item * item;} )

        console.log(result) //[1, 4, 9, 16, 25]
        ```

    - reverse() : 역순으로 재정렬 //arr.reverse();, 배열 자체가 정렬
    - join() : 배열을 합쳐서 문자열을 만듬 ["A", "B" , "C"].join() ⇒ "ABC", join("-") ⇒ "A-B-C"
    - split() : 문자열을 나눠서 배열로
    - isArray() : 배열도, Object도 typeof로 찾으면 Object, 배열임을 확인하려면 isArray() 사용해야 한다. (배열은 Object의 특별한 형태)
    - sort() : 배열 자체가 정렬됨!, 단 기본값으론 문자열 정렬을 사용, Lodash 라이브러리 찾아볼것

        ```jsx
        const arr = [27,8,5,13];
        arr.sort();

        console.log(arr) //[13, 27, 5, 8]

        //비교 함수 직접 만들어줌
        //매개변수로 a,b 받았고 반환 값이 0보다 작은 경우, a,b를 서로 바꿈
        //a,b는 거꾸로 들어온다. 8/27, 5/8.. 순으로 들어옴
        arr.sort((a,b) => {
        	return a-b;
        });

        console.log(arr) // [5, 8, 13, 27]
        ```

    - reduce() : 함수를 순회하며 작업ㅇ르 할 때 사용

        ```jsx
        const arr = [1,2,3,4,5];

        //prev : 누산값, cur: 현재 값 
        const result = arr.reduce((prev,cur) => {
        	return prev + cur;
        }, 0); //초기값

        let userList = [
        	{ name : "A", age : 30},
        	{ name : "B", age : 10},
        	{ name : "C", age : 27},
        	{ name : "D", age : 26},
        	{ name : "E", age : 42},
        	{ name : "F", age : 60},
        ];

        const result2 = userList.reduce((prev,cur) =>{
        	if(cur.age > 19){
        		prev.push(cur.name);
        	}
        	//한바퀴 돌고 나서 prev를 리턴
        	//19세 이상이라면 이름을 push한 값, 아니라면 이전 값 return
        	return prev;
        }, [])
        ```

## 8. 구조 분해 할당 (Destructuring assignment)

- 배열/객체의 속성을 분해해서 값을 변수에 담을 수 있게 하는 표현식

```jsx
//배열 구조 분해 예시
let users = ['Mike', 'Tom', 'Jane'];
let [user1,user2,user3] = users; //user1 = 'Mike', user2 = 'Tom'..

//split을 이용
let str = "Mike-Tom-Jane"
let [user1, user2, user3] = str.split('-');

//값이 없는 경우
let [a,b,c] = [1,2]; //c = undefined

//기본값 사용
let [a=3, b=4, c=5] = [1,2]; 

//일부 반환값 무시
let [user1, , user2] = ['A','B','C','D']; //user1 = 'A', user2 = 'C'

//변수 바꾸기
let a = 1, b =2;
[a,b] = [b,a] //a = 2, b = 1;

//객체 구조 분해 예시
let user = {name : 'Mike', age: 30};
let {name,age} = user;
//변수명을 바꿔서도 사용 가능
let {name : userName, age:userAge} = user; //userName = 'Mike'.. 
```

## 9. 나머지 매개변수, 전개 구문 (Rest parameters, Spread syntax)

- JS 함수에서는, parameter의 제한이 없다.

```jsx
function showName(name){
	console.log(name);
}

showName('Mike'); // Mike
showName('Mike', 'Tom'); // Mike, 'Tom'은 무시된다.

showName(); //undefined 출력됨 
```

- arguments : 함수의 parameter에 접근하는 방법 (ES6 이전에 주로 사용)
    - 배열처럼 length, index 가지고 있지만 Array 내장 메서드(forEach, map 등..) 없는 Array 비슷한 객체로 리턴

```jsx
function showName(name){
	console.log(arguments.length);
	console.log(arguments.[0]);
	console.log(arguments.[1]);
}

showName('Mike', 'Tom');
//2
//'Mike'
//'Tom'
```

- 나머지 매개변수 (Rest parametes) : 정해지지 않은 개수의 인수를 배열로 나타낼 수 있도록 함

```jsx
function showName(...names){
	console.log(names);
}

showName(); //[]
showName('Mike'); // ['Mike']
showName('Mike', 'Tom'); // ['Mike', 'Tom']

//전달받은 인수를 전부 더하는 예제
function add(...numbers){
	let result = 0;
	numbers.forEach((num) => (result+= num));

	return result;
}

//User 객체를 만들어 주는 생성자 함수
//나머지 변수가 가장 마지막 인수임을 기억하기
function User(name,age, ...skills){
	this.name = name;
	this.age = age;
	this.skills = skills;
}
```

- 전개 구문 (Spread syntax)

```jsx
//배열에서의 전개 구문
let arr1 = [1,2,3];
let arr2 = [4,5,6];

let result = [...arr1, ...arr2];

result //1,2,3,4,5,6

//중간에도 사용할 수 있다.
let result = [0, ...arr1, ...arr2, 7,8,9,10] //0,1,2,3...9,10

//Object에서의 전개 구문
let user = {name : 'Mike', age: 30};
let user2 = {...user};
```

## 10. 클로저 (Closure)

- 어휘적 환경 (Lexical Environment)
    - 컴파일러의 Symbol Table 생각해 볼 것
    - 먼저 지역 Lexical Environment(Symbol table 비슷한거) 찾고, 거기에 없다면 외부, 거기 없다면 전역 Lexical Environment로 접근함.

```jsx
function makeAdder(x){
	return function(y){
		return x + y;	
	}
}

const add3 = makeAdder(3);
//add3 함수가 생성된 후에도, 상위함수인 makeAdder의 x에 접근 가능]
//Closure때문에 가능, Closure : 함수와 렉시컬 환경의 조합
//함수가 생성될 당시의 외부 변수 (여기서는 3)을 기억하여 생성 이후에도 접근 가능하게 한다.
console.log(add3(2)); //5

const add10 = makeAdder(10);
console.log(add10(5)); //15
console.log(add3(5)); //8

//이때 num은 직접 접근 불가, 오로지 function 통해서만 접근해야 한다.
//즉, encapsulate가 가능해짐.

function makeCounter(){
	let num = 0;

	return function(){
		return num++;
	}
}

let counter = makeCounter();
counter(); //0
counter(); //1
counter(); //2
```

## 11. setTimeout , setInterval

- setTimeout : 일정 시간이 지난 후 함수를 실행
- setInterval : 일정 시간 간격으로 함수를 반복

```jsx
function fn(){
	console.log(3);
}

//tId : time id
const tId = setTimeout(fn, 3000); //3초당 한번씩 반복 실행, 두번째 인자는 ms

//위와 동일한 코드
setTimeout(function(){
	console.log(3);
},3000);

//Timeout 취소
clearTimeout(tId);

//setInterval도 사용법은 같다..
const tId = setInterval(function(){
	console.log(2)
},3000);

---------------------------------------
//주의! setTimeout의 delay를 0으로 실행해도 순서가 유지되진 않는다.
setTimeout(function(){
	console.log(2)
},0); //delay 0초

console.log(1);

//output : 1 / 2

//이유 : 현재 실행중 script 전부 실행 후, scheduling script 실행된다.
//또한, browser은 평균적으로 4ms정도의 지연시간이 있다.

```

## 12. call, apply, bind

- 함수 호출 방식과 관계없이 this를 지정할 수 있음.

- call : 모든 함수에서 사용 가능, this를 특정값으로 지정할 수 있음.
- apply : call과 같지만, apply는 매개변수를 배열로 받음

```jsx
const mike = { name : "Mike" };

function showThisName(){
	console.log(this.name);
}

showThisName(); //"", 이 때 this는 전역 객체인 window
showThisName.call(mike); // Mike, 이 때 this는 mike

function update(birthYear, occupation){
	this.birthYear = birthYear;
	this.occupation = occupation;
}

//두번째는 birthYear, 세번째는 occupation 
update.call(mike, 1999, "singer");

mike // {name: "Mike", birthYear: 1999, occupation: "singer"}

//apply는 call과 같지만, 배열을 매개변수로 받는다.
update.apply(mike, [2000, "programmar"]);
```

- bind : 함수의 this값을 영구히 바꿀 수 있음

```jsx
function update(birthYear, occupation){
	this.birthYear = birthYear;
	this.occupation = occupation;
}

const updateMike = update.bind(mike)

updateMike(1980, "police");
```

- 종합

```jsx
const user = {
	name : "Mike",
	showName : function(){
		console.log(`hello, ${this.name}`);
	},
}

user.showName(); //hello, Mike

let fn = user.showName;

fn.call(user); //hello, Mike
fn.apply(user); //hello, Mike

let boundFn = fn.bind(user);

boundFn(); //hello, Mike
```

## 13. 상속, prototype

- JS에서도 상속이 있다!

```jsx
const car ={
	wheels: 4,
	drive(){
		console.log("drive...");
	},
};

const bmw = {
	color : "red",
	navigation: true,
};

//car을 상속
bmw.__proto__ = car;

const x5 ={
	color : "white",
	name : "x5", //override
};

//bmw를 상속
x5.__proto__ = bmw;

//이때, object.keys, values는 상속된 프로퍼티는 보여주지 않는다.
Object.keys(x5); //["color","name"]
Object.values(x5); //["white","x5"]

//hasOwnProperty도, 상속받은 프로퍼티는 false를 반환한다.
for(p in x5){
	if(x5.hasOwnProperty(p)){
		console.log('o',p);
	}else{
		console.log('x',p);
	}
}

// color : o, name : o, navigation : x, wheels : x, drive : x
```

- 생성자 함수를 사용하기

```jsx
const Bmw = function(color){
	this.color = color;
}

Bmw.prototype.wheels = 4;
Bmw.prototype.drive = function(){
	console.log("drive..");
};

const x5 = new Bmw("red");
const z4 = new Bmw("blue");

//instanceof 사용하여 어떤 생성자를 통해 만들었는지 알 수 있다
z4 instanceof Bmw // true

z4.constructor === Bmw //true

//위와 같은 코드, 단 constructor를 수동으로 명시해야 한다.
//instanceof 는 사용 가능하다.
Bmw.prototype = {
	//constructor를 수동으로 명시하지 않으면, z4.constructor 사용 불가능하다.
	wheels : 4,
	drive(){
		console.log("drive...");
	}
}

```

## 14. 클래스

- ES6에 추가

```jsx
//기존 생성자 방식
const User = function(name,age){
	this.name = name;
	this.age = age;
	this.showName = function(){
		console.log(this.name);
	};
};

const mike = new User("Mike", 30);

//Class 방식
//이때 name, age는 생성되고
//showName()은 protoyype에 생성된다.
class User2 {
	constructor(name,age){
		this.name = name;
		this.age = age;
	}
	showName(){
		console.log(this.name);	
	}
}

const tom = new User2("Tom", 19);

```

- 생성자 함수는 new 없이도 실행되지만, class는 new 없으면 type error 뜸
- class의 constructor가 class임이 명시됨 (tom.constructor = class User2)

- 클래스에서의 상속 (extends)

```jsx
class Car{
	constructor(color){
		this.color = color;
		this.wheels = 4;
	}
	drive(){
		console.log("drive..");
	}
	stop(){
		console.log("Stop!");
	}
}

class Bmw extends Car{

	constructor(color){
		//반드시 super() 통해 부모의 constructor를 호출해줘야 함.
		//자식이 constructor 만드는 경우, parameter 받아서 super()로 넘겨줘야 함.
		super(color);
		this.navigation = 1;
	}

	drive(){
		//super 키워드 통해 부모 클래스의 메소드를 사용할 수 있음.
		super.drive();
		console.log("BMW drive..");
	}
	park(){
			console.log("Park");
	}
}

const z4 = new Bmw("blue");
```

## 15. 프로미스 (Promise)

- 콜백 함수 : 어떤 작업이 끝났을 때 실행되는 함수!

```jsx
const pr = new Promise((resolve, reject) => { });

//resolve : 성공할때 호출되는 콜백 함수
//reject : 실패할때 호출되는 콜백 함수
```

- Promist Object의 property
    - state : pending(대기, 초기 실행)
        - 만약 성공 (resolve) 호출되면 : fulfilled (이행됨)
        - 만약 실패 (reject) 호출되면 : rejected(거부됨)
    - result : undefined (초기값), 이후 resolve, reject 호출되면 변경됨

```jsx
//3초 뒤 성공하는 promise, 3초 뒤 state: fulfilled, result : 'OK'로 변경
const pr = new Promise((resolve, reject) => {
	setTimeout(()=>{
		resolve('OK')
	},3000);
 });

//3초 뒤 실패하는 promise, 3초 뒤 state: rejected, result : error로 변경
const pr = new Promise((resolve, reject) => {
	setTimeout(()=>{
		reject(new Error('error..'))
	},3000);
 });

```

- customer는 then을 이용하여 resolve, reject 처리 가능하다.

```jsx
const pr = new Promise((resolve, reject) => {
	setTimeout(()=>{
		resolve('OK')
	},3000);
 });

pr.then(
	//Promise가 성공했을때 실행, result에는 result값이 들어감.
	//이 경우 'OK' 들어온다.
	function(result){
		console.log(result);
	},
	function(err){
		console.log(err);
	}
);

//catch를 통해서도 사용할 수 있다.
//위 코드와 아래 코드는 동일! (finally 제외)
//이때, 만약 성공 함수에서 error 발생했다면 error 함수가 catch 해 주므로
//위 코드보단 아래 코드를 사용하는 것이 권장된다.
pr.then(
		function(result){}
).catch(
		function(err){
		console.log(err);
	}
).finally(
		function(){
			//로딩 화면을 없애는 등에 사용할 수 있다
			console.log('이 내용은 항상 실행된다'); 
		}
)
```

- Promise 없이 callback으로 구현하는 경우, CALLBACK으로 인해 코드의 Depth가 매우 깊어진다.

```jsx
//2초 간격으로 연달아서 f1,f2,f3 실행

const f1 = (callback) =>{
	setTimeout(function(){
		console.log("f1 실행 끝!");
		callback();
	},2000);
};

const f2 = (callback) =>{
	setTimeout(function(){
		console.log("f2 실행 끝!");
		callback();
	},2000);
};

const f3 = (callback) =>{
	setTimeout(function(){
		console.log("f3 실행 끝!");
		callback();
	},2000);
};

console.log('시작');
f1(function(){
	f2(function(){
		f3(function(){
			console.log('끝')
		});
	});
});
```

- 같은 코드를 Promise를 통해 작성하면..

```jsx
//1,2,3초 간격으로 연달아서 f1,f2,f3 실행

const f1 = () =>{
	return new Promise((res,rej) =>{
		setTimeout(()=>{
			res("f1 실행 끝!");
		},1000);
	});
};

const f2 = (message) =>{
	console.log(message);
	return new Promise((res,rej) =>{
		setTimeout(()=>{
			res("f2 실행 끝!");
		},2000);
	});
};

const f3 = (message) =>{
	console.log(message);
	return new Promise((res,rej) =>{
		setTimeout(()=>{
			res("f3 실행 끝!");
		},3000);
	});
};

//이렇게 Promise가 연결되는 것을 promise chaining이라고 한다!
//이런 경우 순차적으로 실행, 총 실행시간은 6초
console.time('Promise Chaining');
console.log('시작');
f1()
.then((res)=> f2(res))
.then((res)=> f3(res))
.then((res)=>console.log(res))
.catch(console.log)
.finally(()=>{
    console.log('끝');
    console.timeEnd('Promise Chaining');
});

//Promise.all을 사용하여 동시에 실행하기
//배열 안의 promise가 모두 실행되어야 .then 파트가 실행된다.
//이 경우 동시 실행, 총 실행시간은 3초 (가장 긴 Promise 시간)
//만약에 하나라도 Reject 되면, 다른 Promise 성공하더라도 Error만 내뱉는다.
console.time('Promise.all')
Promise.all([f1(), f2(), f3()])
.then((res)=>{
    console.log(res);
    console.timeEnd('Promise.all')
});

//Promise.race를 사용하여 가장 빨리 실행된 Promise만 실행하기
//하나라도 Promise가 완료되었을 경우, 해당 Promise값을 리턴하고 끝난다.
//이 경우 동시 실행, 총 실행시간은 1초 (가장 짧은 Promise 시간)
//만약 이때  f1(1초) 가 reject라면, error 반환하고 종료된다. (가장 빠르기 때문)
//만약 이때 f2(2초), f3(3초)가 reject라면, f1의 resolve 반환하고 종료되고 f2,f3 에러는 무시된다.
//(f1이 먼저 실행되고 종료되므로)
//대용량 이미지 등, 하나라도 다운받았을때 바로 보여주는 코드 등에 사용
console.time('Promise.race')
Promise.race([f1(), f2(), f3()])
.then((res)=>{
    console.log(res);
    console.timeEnd('Promise.race')
}); // 리턴값 : f1 실행 끝!
```

## 16. async, await

- async를 붙이면 항상 Promise를 반환한다.

```jsx
//기본적으로 Promise를 반환한다
async function getName(){
	return "Mike";
}

getName(); // Promise {<fulfilled>: "Mike"}

//Promise를 반환하면, 해당 Promise를 그대로 사용한다.
async function getName(){
	return Promise.resolve('Mike');
}

getName(); // Promise {<fulfilled>: "Mike"}

//만약 함수 내부에서 에러가 발생하면, reject된 Promise를 반환한다.
async function getName(){
	throw new Error('Error...');
}

//Promise {<rejected>: Error: Error...
//at getName (<anonymous>:2:8)
//at <anonymous>:1:1}
```

- await 키워드는 async 함수 내부에서만 사용 가능하다.

    await 키워드는 프로미스를 반환하는 함수 앞에 오며, Promise가 반환될때까지 기다린다.

```jsx
function getName(name){
    console.log('getName 호출');
    return new Promise((resolve,reject)=>{
        setTimeout(()=>{
            resolve(name);
        },1000);
    });
}

async function showName(){
	const result = await getName('Mike');
    console.log(result);
}

console.log('시작');
showName();
```

- 위의 Promise Chaining을 async, awiat으로 변환

```jsx
const f1 = () =>{
	return new Promise((res,rej) =>{
		setTimeout(()=>{
			res("f1 실행 끝!");
		},1000);
	});
};

const f2 = (message) =>{
	console.log(message);
	return new Promise((res,rej) =>{
		setTimeout(()=>{
			res("f2 실행 끝!");
		},2000);
	});
};

const f3 = (message) =>{
	console.log(message);
	return new Promise((res,rej) =>{
		setTimeout(()=>{
			res("f3 실행 끝!");
		},3000);
	});
};

//await을 사용하면 가독성이 더 좋다.
console.log('시작');
async function callFunction(){
    const result1 = await f1();
    const result2 = await f2(result1);
    const result3 = await f3(result2);
    console.log(result3);
    console.log('끝');
}

callFunction();

//아래 코드와 위 코드는 동일하다
console.log('시작');
f1()
.then((res)=> f2(res))
.then((res)=> f3(res))
.then((res)=>console.log(res))
.catch(console.log)
.finally(()=>{
    console.log('끝');
});

//------------------------------------

//만약 중간에 reject 발생한다면, try-catch문 사용 
console.log('시작');
async function callFunction(){
		try{
    const result1 = await f1();
    const result2 = await f2(result1);
    const result3 = await f3(result2);
    console.log(result3);
		}catch (e){
			console.log(e);
		}
    console.log('끝');
}

//Promise.all등에서도 사용할 수 있다.
const result = await Promise.all([f1,f2,..);
```

## 17. Generator

- 함수의 실행을 중간에 멈췄다가 재개할 수 있는 기능

```jsx
//function 옆에 * 사용하여 Generator 함수 실행
//yield에서 함수를 멈출 수 있다.
function* fn(){
    console.log(1);
    yield 1;
    console.log(2);
    yield 2;
    console.log(3);
    console.log(4);
    yield 3;
    return "finish";
}

//함수가 실행되지 않고 Generated된 값을 반환한다.
const a = fn();

a // fn {<suspended>}

//가장 가까운 yield까지 실행한다.
//지금의 경우 yield 1까지 실행하게 된다.
a.next();
// 1 출력, {value: 1, done: false}
//value : yield값, done : 함수가 끝났는지 여부
//만약 yield 오른쪽의 값을 생략하면 undefined가 들어간다.

a.next();
// 2 출력, {value: 2, done: false}

a.next();
// 3,4 출력, {value: 3, done: false}

a.next();
// {value: "finish", done: true}
// value에는 return값을, 함수가 끝났으므로 done은 true를 반환한다.

a.next();
//return 이후에 실행하면 {value: undefined, done: true}
//return 값은 없으므로 undefined, done은 여전히 true를 반환한다.
```

- return() 메소드

```jsx
//위 코드에서 중간에
a.return('END'); // {value: "END", done: true}
```

- throw() 메소드

```jsx
//Try-catch문으로 감싼 뒤
function* fn(){
    try{
        console.log(1);
        yield 1;
        console.log(2);
        yield 2;
        console.log(3);
        console.log(4);
        yield 3;
        return "finish";
    }catch (e){
        console.log(e);
    }
}

const a = fn();
a.next();

//실행 중 Error를 throw하면 done : true가 된다.
a.throw(new Error('ERROR!!'));

//Error: ERROR!!
//    at <anonymous>:1:9
// {value: undefined, done: true}
```

- Iterator과 iterable
    - Generator는 iterable(반복 가능)
        - Symbol.iterator라는 메서드가 있어야 한다.
        - Symbol.iterator는 iterator를 반환해야 한다.

    - Iterator
        - Next 메서드를 가진다.
        - next 메세지는 value, done 속성을 가진 객체를 반환한다.
        - 작업이 끝나면 done은 true가 된다.

```jsx
//배열은 Prototype에 Symbol.iterator가 구현되어 있다.
//따라서 배열은 Iterable한 객체이다.
const arr = [1,2,3];
const it = arr[Symbol.iterator]();

it.next(); // {value: 1, done: false}
it.next(); // {value: 2, done: false}
it.next(); // {value: 3, done: false}
it.next(); // {value: undefined, done: true}

//배열이 iterable하므로, for of문을 사용할 수 있다.
for(let num of arr){
	console.log(num);
};

function* fn(){
	yield 4;
	yield 5;
	yield 6;
}

const a = fn();

//마찬가지로 Generator도 Iterable하다.
for(let num of a){
	console.log(num);
} //4,5,6 출력
```

- Generator에 Paramenter 전달

```jsx
function* fn(){
    const num1 = yield "첫번째 숫자를 입력하세요.";
    console.log(num1);

    const num2 = yield "두번째 숫자를 입력하세요.";
    console.log(num2);

    return num1 + num2;
}

const a = fn();

a.next(); //{value: "첫번째 숫자를 입력하세요.", done: false}

a.next(1); // 1 출력, {value: "두번째 숫자를 입력하세요.", done: false}

a.next(2); // 2 출력, {value: 3, done: true}
```

- Generator는 값을 미리 만들어두지 않고, 필요할 때 계산한다.

```jsx
//따라서 아래와 같은 코드도, 필요한 만큼만 실행하므로 무한 Loop 걸리지 않는다.
function* fn(){
    let index = 0;
    while(true){
        yield index++;
    }
}

const a = fn();

a.next(); //{value: 0, done: false}
a.next(); //{value: 1, done: false}
a.next(); //{value: 2, done: false}

```