# React : Props와 State

분류: React
작성일시: 2021년 7월 8일 오후 2:32

- 요약
    1. Props는 부모에서 자식으로 보내는 값 (readOnly), State는 컴포넌트 내부에 있는 변수
    2. State 바꿀 때 setState 사용하지 않는다면 re-render 일어나지 않으므로 값이 바뀌어도 보이지 않는다.

## 1. Props

- 부모 컴포넌트가 자식 컴포넌트에게 넘겨주는 값

루트 폴더에 App.js와 Myname.js 있을 때

- App.js

```jsx
import React, { Component } from 'react';
import MyName from './Myname'; //import

class App extends Component {
  render() {
    return <MyName name="리액트" />; //props로 값 전달
  }
}

export default App;
```

- Myname.js

```jsx
import React, { Component } from 'react';

class MyName extends Component {
  render() {
    return (
      <div>
        안녕하세요! 제 이름은 <b>{this.props.name}</b>입니다.
        {/* props로 값 받아옴 */}
      </div>
    );
  }
}

export default MyName;
```

만약, 기본값 props 사용하고 싶다면?

- Myname.js

```jsx
import React, { Component } from 'react';

class MyName extends Component {
  
  //기본 Props 설정
  static defaultProps = {
    name: '기본이름'
  };

  render() {
    return (
      <div>
        안녕하세요! 제 이름은 <b>{this.props.name}</b>입니다.
        {/* props로 값 받아옴 */}
      </div>
    );
  }
}

export default MyName;
```

혹은

```jsx
import React, { Component } from 'react';

class MyName extends Component {
  render() {
    return (
      <div>
        안녕하세요! 제 이름은 <b>{this.props.name}</b>입니다.
        {/* props로 값 받아옴 */}
      </div>
    );
  }
}

MyName.defaultProps = {
  name: 'lemon'
};

export default MyName;
```

위와 아래는 같은 코드이나, 위쪽 코드가 좀 더 최신 방법

또한, [비구조화 할당 문법](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)을 이용하여 다음과 같이 작성도 가능하다.

```jsx
import React from 'react';
//Component 더 이상 불러오지 않아도 됨

const MyName = ({ name }) => {
  return <div> 안녕하세요! 제 이름은 {name} 입니다. </div>;
};

MyName.defaultProps = {
  name: 'lemon'
};

export default MyName;
```

함수형 컴포넌트 : State, LifeCycle 없다.

대신, 초기 Mount 속도 빠르고 Memory도 덜 사용함. 따라서, 단순히 보이기만 하는 부분은 함수형 컴포넌트 사용하면 좋다.

하지만 엄청나게 많은 Component 만드는거 아니라면 성능 차이는 미미

## 2. State

- Component 자기 자신이 들고 있음.
- State는 내부에서 변경할 수 있다.
- 변경할 때는 언제나 setState() 라는 함수를 사용한다.

- Counter.js

```jsx
import React, { Component } from 'react';

class Counter extends Component {
  state = {
    number: 0
  };

  handleIncrease = () => {
		//State 바꿀 땐 setState 함수 통해서 바꿔야 한다.
		//(setState 함수를 통해야 Component가 State가 변했다는걸 알 수 있음)
    this.setState({
      number: this.state.number + 1
    });
  };

  handleDecrease = () => {
    this.setState({
      number: this.state.number - 1
    });
  };

  render() {
    return (
      <div>
        <h1>카운터</h1>
        <div> 값 : {this.state.number} </div>
        <button onClick={this.handleIncrease}>+</button>
        <button onClick={this.handleDecrease}>-</button>
      </div>
    );
  }
}

export default Counter;
```

- 만약, handleIncrease, handleDecrease 가 Arrow Function 아니면 this 키워드 사용할 수 없다. (undefined로 나옴)

```jsx
constructor(props) {
    super(props);
    this.handleIncrease = this.handleIncrease.bind(this);
  }

  handleIncrease() {
    this.setState({
      number: this.state.number + 1
    });
  }
```

- 만약 Arrow Function 사용하지 않고 싶다면, Constructor 사용해서 구현할 수 있다.