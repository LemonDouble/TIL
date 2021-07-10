# React : LifeCycle API

분류: React
작성일시: 2021년 7월 10일 오후 1:46

## 1. LifeCycle API

- 참고자료 : [https://react-anyone.vlpt.us/05.html](https://react-anyone.vlpt.us/05.html)

- 컴포넌트가
    1. 나타날 때
    2. 업데이트 될 때
    3. 사라질 때

    어떤 액션을 하고 싶다면, LifeCycle API를 사용

- 용어 정리
    - Mounting : Component가 Browser상에 나타나는 것
    - Updating : Component의 Props나 State가 바뀌었을 때
    - Unmounting : Component가 Browser 상에서 사라질 때

    ![React%20LifeCycle%20API%20a66fd819f11e44f2b5e36b40cbdcbd10/Untitled.png](React%20LifeCycle%20API%20a66fd819f11e44f2b5e36b40cbdcbd10/Untitled.png)

---

### 1. 컴포넌트 초기 생성

- Constructor : 생성자, Component 만들어 질 때마다 호출됨

```jsx
constructor(props) {
    super(props);
    console.log('constructor');
  }
```

- componentDidMount : 컴포넌트가 화면에 나타나게 했을 때 호출

```jsx
componentDidMount() {
  // 외부 라이브러리 연동: D3, masonry, etc
  // 컴포넌트에서 필요한 데이터 요청: Ajax, GraphQL, etc
  // DOM 에 관련된 작업: 스크롤 설정, 크기 읽어오기 등
}
```

//  ref : ID와 비슷, DOM의 정보를 가져올 수 있다

- 특정 DOM의 render 정보를 알아보는 경우 :

```jsx
class App extends Component {
  componentDidMount() {
		//DOM이 Mount 된 후에 위치를 알아야 하므로 componentDidMount 쓴다
    console.log('componentDidMount');
    console.log(this.myDiv.getBoundingClientRect());
		// = DOMRect {x: 8, y: 21.4375, width: 778, height: 43, top: 21.4375…}
  }

  render() {
    return (
			//해당 div 데이터를 ref로 넘긴다.
      <div ref={(ref) => (this.myDiv = ref)}>
        <h1> Hello, React! </h1>
      </div>
    );
  }
}
```

### 2. 컴포넌트 업데이트

- getDerivedStateFromProps() : props로 받아온 값을 state로 동기화하는 작업을 하는 경우

```jsx
static getDerivedStateFromProps(nextProps, prevState) {
  // 여기서는 setState 를 하는 것이 아니라
  // 특정 props 가 바뀔 때 설정하고 설정하고 싶은 state 값을 리턴하는 형태로
  // 사용됩니다.
  /*
  if (nextProps.value !== prevState.value) {
    return { value: nextProps.value };
  }
  return null; // null 을 리턴하면 따로 업데이트 할 것은 없다라는 의미
  */
}
```

MyComponent.js에서 App.js에서 받아온 Props를 state와 동기화하는 예제

- App.js (Click하면 App.js 내의 State가 1씩 증가, 해당 counter state를 props로 Mycomponent에 전달)

```jsx
import React, { Component } from 'react';
import MyComponent from './MyComponent';

class App extends Component {
  state = {
    counter: 1
  };

  handleClick = () => {
    this.setState({
      counter: this.state.counter + 1
    });
  };

  render() {
    return (
      <div>
        <MyComponent value={this.state.counter} />
        <button onClick={this.handleClick}> Click Me! </button>
      </div>
    );
  }
}

export default App;
```

- MyComponent.js ( getDerivedStateFromProps 로 Props와 State를 동기화)

```jsx
import React, { Component } from 'react';

class MyComponent extends Component {
  state = {
    value: 0
  };

  static getDerivedStateFromProps(nextProps, prevState) {
    //nextProps : 받아올 Props, prevState : 현재 State

    if (prevState.value !== nextProps.value) {
      return {
        value: nextProps.value
      };
    }
    return null;
  }

  render() {
    return (
      <div>
        <p> props : {this.props.value} </p>
        <p> state : {this.state.value} </p>
      </div>
    );
  }
}

export default MyComponent;
```

- ShouldComponentUpdate() : 컴포넌트 최적화할떄 사용, React는 virtual DOM 먼저 그려서 직접 DOM 그리는것보단 빠르지만, Vritual DOM 렌더링도 자원을 소모한다.
- 현재 컴포넌트 상태가 업데이트되지 않아도 부모 컴포넌트가 리렌더링되면 자식 컴포넌트도 Virtual DOM에 리렌더링된다. 이를 방지하기 위해서 사용 가능
- False 반환하면 render 함수 호출하지 않음

```jsx
shouldComponentUpdate(nextProps, nextState) {
  // return false 하면 업데이트를 안함
  // return this.props.checked !== nextProps.checked
  return true;
}
```

- Mycomponent.js에서 다음 코드 추가하면, 10일때 render 되지 않음

```jsx
shouldComponentUpdate(nextProps, nextState) {
    if (nextProps.value === 10) return false;
    return true;
  }
```

- getSnapshotBeforeUpdate() : Component가 Browser의 DOM에 반영되기 직전에 호출됨

업데이트 되기 직전의 DOM 상태를 반환하고, 이후 componentDidUpdate() 에서 3번째 파라미터로 받아올 수 있음

Chrome은 Scroll 위치 유지가 기본 구현되어있지만, 다른 브라우저에서도 같은 기능을 구현하기 위해 사용할 수 있다.

[https://codesandbox.io/s/484zvr87ow](https://codesandbox.io/s/484zvr87ow)  << Scroll 유지 구현 코드

```jsx
getSnapshotBeforeUpdate(prevProps, prevState) {
    // DOM 업데이트가 일어나기 직전의 시점입니다.
    // 새 데이터가 상단에 추가되어도 스크롤바를 유지해보겠습니다.
    // scrollHeight 는 전 후를 비교해서 스크롤 위치를 설정하기 위함이고,
    // scrollTop 은, 이 기능이 크롬에 이미 구현이 되어있는데, 
    // 이미 구현이 되어있다면 처리하지 않도록 하기 위함입니다.
    if (prevState.array !== this.state.array) {
      const {
        scrollTop, scrollHeight
      } = this.list;

      // 여기서 반환 하는 값은 componentDidMount 에서 snapshot 값으로 받아올 수 있습니다.
      return {
        scrollTop, scrollHeight
      };
    }
  }

	//실제로 DOM에 업데이트 된 후에 호출
  componentDidUpdate(prevProps, prevState, snapshot) {
    if (snapshot) {
      const { scrollTop } = this.list;
      if (scrollTop !== snapshot.scrollTop) return; // 기능이 이미 구현되어있다면 처리하지 않습니다. (Chrome)
      const diff = this.list.scrollHeight - snapshot.scrollHeight;
      this.list.scrollTop += diff;
    }
  }
```

- componentDidUpdate() : render() 호출 뒤 발생, this.props와 this.state 가 이미 바뀌어 있으므로, 해당 값을 통해 변화를 확인할 수 있다.

```jsx
componentDidUpdate(prevProps, prevState, snapshot) {
    if (this.props.value !== prevProps.value) {
      console.log('value 값 변경!', this.props.value);
    }
  }
```

### 3. 컴포넌트 제거

- componentWillUnmount() : Component가 사라질 때 호출

```jsx
componentWillUnmount() {
  // 이벤트, setTimeout, 외부 라이브러리 인스턴스 제거
}
```

### 4. 컴포넌트에 에러가 발생한 경우

- componentDidCatch() : 컴포넌트의 에러가 발생한 경우 호출,
- 네트워크 통해 Error을 특정 서버로 보내거나, Error 발생 창을 띄우거나 등

```jsx
componentDidCatch(error, info) {
  this.setState({
    error: true
  });
}
```

componentDidCatch는, Error가 발생할 수 있는 부모 Component에서 호출해야 한다.