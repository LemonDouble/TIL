# React : 배열에서 데이터 제거하기

분류: React
작성일시: 2021년 7월 13일 오후 2:51

### 1. Slice, Filter

배열에서 데이터를 제거 수정할 때는 >> 불변성 << 을 유지해야 한다.

따라서 .slice, 혹은 .filter를 사용한다.

- Slice : [https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/slice](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/slice)

    ```jsx
    const animals = ['ant', 'bison', 'camel', 'duck', 'elephant'];

    console.log(animals.slice(2));
    // expected output: Array ["camel", "duck", "elephant"]

    console.log(animals.slice(2, 4));
    // expected output: Array ["camel", "duck"]

    // 추가 사용법
    const numbers = [1,2,3,4,5];

    numbers.slice(0,2).concat(numbers.slice(3,5))
    // [1,2,4,5]

    [
    	...numbers.slice(0,2),
    	10,
    	...numbers.slice(3,5)
    ]
    // [1,2,10,4,5]
    ```

- Filter : [https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/filter](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/filter)

    ```jsx
    const words = ['spray', 'limit', 'elite', 'exuberant', 'destruction', 'present'];

    const result = words.filter(word => word.length > 6);

    console.log(result);
    // expected output: Array ["exuberant", "destruction", "present"]

    //추가 사용법
    const numbers = [1,2,3,4,5];

    numbers.filter(n => n> 3);
    //[4,5]

    numbers.filter(n => n !==  3);
    //[1,2,4,5]
    ```

### 2. 코드

1. App.js에 handleRemove 함수를 추가하고, 내려준다

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

	//handleRemove 함수를 추가
  handleRemove = (id) =>{
    const {information} = this.state;

    this.setState({
      information : information.filter(info => info.id !== id)
    });
  }
  
  render() {
    return (
      <div>
        <PhoneForm onCreate={this.handleCrate}/>
        <PhoneInfoList 
          data={this.state.information}
          onRemove={this.handleRemove}
        />
      </div>
    );
  }
}

export default App;
```

2.