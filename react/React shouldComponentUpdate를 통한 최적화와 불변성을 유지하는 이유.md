# React : shouldComponentUpdate를 통한 최적화와 불변성을 유지하는 이유

분류: React
작성일시: 2021년 7월 13일 오후 8:22

## 1. shouldComponentUpdate를 통한 최적화

기본적으로, 설정하지 않는다면 shouldComponentUpdate는 항상 true를 반환한다.

```jsx
shouldComponentUpdate(nextProps, nextState) {
	return true;
}
```

하지만 이런 경우, state, props가 변경되지 않았을 때도 다시 렌더링되므로 비효율적이다.

따라서, PhoneInfo.js에서

```jsx
shouldComponentUpdate(nextProps, nextState) {
        // 만약 State 변경된다면 업데이트 한다.
        if(this.state !== nextState){
            return true;
        }

        //Info 또한 다를때만 업데이트 한다.
        return this.props.info !== nextProps.info;
    }
```

와 같이 작성하여 State나 Props(Info) 가 변경됐을 때만 다시 렌더링되도록 바꿀 수 있다.

## 2. 불변성

1. 불변성을 유지하지 않았을 경우,

    Reference Type이므로 다음과 같은 문제가 발생한다.

    따라서, 두 오브젝트가 다른지 확인하기 위해선 내부에 있는 값을 확인해야 한다.

    (Object도 마찬가지)

```jsx
const array = [0,1,2];
const anotherArray = array;
array.push(3);

anotherArray
//[0,1,2,3]

array
//[0,1,2,3]

array === anotherArray
//true
```

2. 불변성을 유지한 경우, 객체간의 비교가 간편하다

```jsx
const array = [0,1,2];
const anotherArray = [...array, 3];

array
//[0,1,2]

anotherArray
//[0,1,2,3]

array === anotherArray
//false

array !== anotherArray
//true
```

- 하지만 불변성을 유지하는 경우, 다음과 같은 복잡한 오브젝트등을 불변성을 유지한 채로 작업하기 힘들 수 있다.  이런 경우
    - [https://github.com/immerjs/immer](https://github.com/immerjs/immer) ( immer)
    - [https://immutable-js.com/](https://immutable-js.com/) (Immutablejs)

    등을 사용할 수 있다.