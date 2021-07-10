# React : 배열에 데이터 삽입하기

분류: React
작성일시: 2021년 7월 10일 오후 6:47

## 1. 배열에 데이터 삽입하기

- PhoneForm.js에 다음 코드 추가

```jsx
handleSubmit = (e) =>{
        //원래 해야되는 Action을 하지 않도록 한다.
        //Submit의 경우는 새로고침 되는 것을 방지
        e.preventDefault();

        this.props.onCreate({
            name: this.state.name,
            phone: this.state.phone
        })
    }

render() {
        return (
            <form onSubmit={this.handleSubmit}> // << 수정!
```

- App.js를 다음과 같이 수정

```jsx
import React, { Component } from 'react';
import PhoneForm from './components/PhoneForm';

class App extends Component {

  handleCrate = (data) => {
    console.log(data);
		//{name: "ㅁㅁ", phone: "ㅇㅇ"} 와 같이 출력
  }
  
  render() {
    return (
      <div>
        <PhoneForm onCreate={this.handleCrate}/>
      </div>
    );
  }
}

export default App;
```

위와 같이 작성시, 자식의 state가 부모에게 전달된다.

---

이제 handleCreate에서 받아온 값으로,

```jsx
state = {
    information : [],
}
```

다음과 같이 information 배열에 값을 수정하려고 할 때

### React에서는 불변성(Immutable) 을 유지해야 한다

불변성 : 기존 배열, 값을 수정하지 않고 새로운 객체, 값을 만들어 값을 주입해줘야 함

따라서,

```jsx
handleCrate = (data) => {
		//setState 사용되지 않아 Component가 State 바뀐것을 알 수 없고,
		//push는 불변성 유지가 불가능하다. (x) 
    this.state.information.push(data);

// -------------------------------------------------------

    this.setState({
		//(O)
      information : this.state.information.concat(data)
    })
  }
```

또한, 값을 관리하기 위한 Unique한 ID를 추가할 수 있다.

추가하는 방법은 Spread(전개 구문) 을 사용한 방법과, Object.assign 함수를 사용할 수 있다.

- Spread (전개 구문) 사용 ([https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/Spread_syntax](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/Spread_syntax))

```jsx
id = 0;
  
state = {
  information : [],
}

handleCrate = (data) => {
    this.setState({
      information : this.state.information.concat({
        id: this.id++,
        // spread 전개 연산자 문법
        ...data,
      })
    })
  }

//{"id":0,"name":"aaa","phone":"ddd"}
```

- Object.assign 사용

    ([https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object/assign](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object/assign))

```jsx
handleCrate = (data) => {
    this.setState({
      information : this.state.information.concat(Object.assign({},{id: this.id++},data))
    })
  }

//{"id":0,"name":"aaa","phone":"ddd"}
```

이때, id를 State에 넣어주지 않은 이유는, State는 Render할때 사용하는데 id는 render하는데 사용하지 않으므로 굳이 State에 넣어주지 않아도 된다.

---

## 전체 코드

- App.js

```jsx
import React, { Component } from 'react';
import PhoneForm from './components/PhoneForm';

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
        {JSON.stringify(this.state.information)}
      </div>
    );
  }
}

export default App;
```

- PhoneForm.js

```jsx
import React, { Component } from 'react';

class PhoneForm extends Component {

    state = {
        name : '',
        phone : '',
    }

    //e = 이벤트 객체
    handleChange = (e) =>{
        this.setState({
            [e.target.name] : e.target.value
        });
    }

    handleSubmit = (e) =>{
        //원래 해야되는 Action을 하지 않도록 한다.
        //Submit의 경우는 새로고침 되는 것을 방지
        e.preventDefault();

        this.props.onCreate({
            name: this.state.name,
            phone: this.state.phone
        })

        this.setState({
            name : '',
            phone : '',
        })
    }

    render() {
        return (
            <form onSubmit={this.handleSubmit}>
                <input placeholder= "이름" onChange={this.handleChange} value ={this.state.name} name ="name"/>
                <br/>
                <input placeholder="전화번호" onChange={this.handleChange} value={this.state.phone} name="phone" />
                <br/>
                <button type="submit">등록</button>
            </form>
        );
    }
}

export default PhoneForm;
```