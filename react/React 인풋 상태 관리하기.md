# React : 인풋 상태 관리하기

분류: React
작성일시: 2021년 7월 10일 오후 4:46

- Create-react-app을 yarn start로 시작한 경우, 해당 서버는 개발용 서버이므로 코드 수정 내용이 즉시 반영된다. (서버 끌 필요 없음!)

## 1. 인풋 상태 관리

- App.js

```jsx
import { useState } from 'react';
import './App.css';
import PhoneForm from './components/PhoneForm';

function App() {

  return (
    <div>
      <PhoneForm />
    </div>
  );
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

    render() {
        return (
            <form>
                <input placeholder= "이름" onChange={this.handleChange} value ={this.state.name} name ="name"/>
                <br/>
                <input placeholder="전화번호" onChange={this.handleChange} value={this.state.phone} name="phone" />
                <br/>
                <div>
                    {this.state.name}
                    {this.state.phone}
                </div>
            </form>
        );
    }
}

export default PhoneForm;
```

e 부분 문법 (  [e.target.name] : e.target.value ) : 계산된 속성명(Computed Property Names)

Form에서 change 발생할 경우, onChange의 핸들러로 handleChange를 선언했으므로, handleChange 함수가 호출된다.

이후, handleChange 함수는 state를 바꾸는데, target 이름 : target value로 state를 변경한다.

- 참고자료 : [https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/Object_initializer](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/Object_initializer)