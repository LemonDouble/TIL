# React : 배열 렌더링하기

분류: React
작성일시: 2021년 7월 13일 오후 2:36

### 1. Javascript 배열 내장함수 map

- 참고자료 : [https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/map](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/map)

```jsx
const array1 = [1, 4, 9, 16];

// pass a function to map
const map1 = array1.map(x => x * 2);

console.log(map1);
// expected output: Array [2, 8, 18, 32]

```

- 각 원소에 대해 각각 해당 함수를 적용시키는 함수이다.

### 2. 데이터 전달하고 렌더링하기

src/PhoneInfo.js 추가 : 제일 마지막에 렌더링하는 컴포넌트

```jsx
import React, { Component } from 'react';

class PhoneInfo extends Component {
    render() {
        const {name, phone, id} = this.props.info;
				//비구조화 할당을 통해 name, phone, id 받아옴

        const style ={
            border : '1px solid black',
            padding : '8px',
            margin:'8px',
        };

        return (
            <div style={style}>
                <div><b>{name}</b></div>
                <div>{phone}</div>
            </div>
        );
    }
}

export default PhoneInfo;
```

src/PhoneInfoList.js 추가

```jsx
import React, { Component } from 'react';
import PhoneInfo from './PhoneInfo';

class PhoneInfoList extends Component {

    static defaultProps = {
        data: []
    }
    //만약 Props 없으면 기본 Props로 빈 배열 사용
    
    render() {
        const {data} = this.props;
        const list = data.map(
            info => (<PhoneInfo info={info} key={info.id} />)
        );
        //info를 PhoneInfo로 변환 및 전달, key는 컴포넌트 렌더링할때 고유값을 정해줘 업데이트 성능을 최적화
				
        return (
            <div>
                {list}
            </div>
        );
    }
}

export default PhoneInfoList;
```

App.js

```jsx
import React, { Component } from 'react';
import PhoneForm from './components/PhoneForm';
import PhoneInfoList from './components/PhoneInfoList';

class App extends Component {

  id = 0;
  
  state = {
    information : [],
  }

  handleCrate = (data) => {
    this.setState({
      information : this.state.information.concat(Object.assign({},{id: this.id++},data))
    })
  }
  
  render() {
    return (
      <div>
        <PhoneForm onCreate={this.handleCrate}/>
        <PhoneInfoList data={this.state.information}/> //Information 배열 전달
      </div>
    );
  }
}

export default App;
```

### 3. 왜 Key를 사용하는가?

- 참고자료 : [https://ko.reactjs.org/docs/lists-and-keys.html](https://ko.reactjs.org/docs/lists-and-keys.html)
- 참고자료 2 : [https://ko.reactjs.org/docs/reconciliation.html#recursing-on-children](https://ko.reactjs.org/docs/reconciliation.html#recursing-on-children)

- Key 존재하지 않는 경우

```jsx
<ul>
  <li>first</li>
  <li>second</li>
</ul>

<ul>
  <li>first</li>
  <li>second</li>
  <li>third</li>
</ul>
```

의 경우( 가장 끝에 추가) 리액트는 DOM 트리를 비교하고, third 자식을 추가하므로 문제 없지만

```jsx
<ul>
  <li>Duke</li>
  <li>Villanova</li>
</ul>

<ul>
  <li>Connecticut</li>
  <li>Duke</li>
  <li>Villanova</li>
</ul>
```

의 경우 (가장 처음에 추가),

리액트는 Connecticut 과 Duke를 비교한 뒤, Duke를 Connecticut으로 변경,

Villanova와 Duke를 비교한 뒤, Villanova를 Duke로 변경

Villianova를 추가

하는 식으로 매우 비효율적으로 동작한다.

- Key 있는 경우

```jsx
<ul>
  <li key="2015">Duke</li>
  <li key="2016">Villanova</li>
</ul>

<ul>
  <li key="2014">Connecticut</li>
  <li key="2015">Duke</li>
  <li key="2016">Villanova</li>
</ul>
```

리액트가 키를 통해 각 노드를 구별할 수 있으므로, 단지 Connecticut을 추가한다.

Key가 존재할 때 효율적으로 작동한다!