# React : Ref - DOM에 직접 접근하는 법

분류: React
작성일시: 2021년 7월 13일 오후 9:25

## 1. ref의 사용 방법 (1)

- name, Phone을 입력하고 난 후에, focus를 (커서를) name으로 돌리고 싶다면?

```jsx
import React, { Component } from 'react';

class PhoneForm extends Component {
		// (1) input이란 변수를 선언
    input = null;

    state = {
        name : '',
        phone : '',
    }

    handleChange = (e) =>{
        this.setState({
            [e.target.name] : e.target.value
        });
    }

    handleSubmit = (e) =>{

        e.preventDefault();

        this.props.onCreate({
            name: this.state.name,
            phone: this.state.phone
        })

        this.setState({
            name : '',
            phone : '',
        })

				//(3) Submit 후에, 받아온 ref 이용해서 input form에 접근한 후,
				//    Focus 통해 커서를 옮겨준다.
        this.input.focus();
    }

    render() {
        return (
            <form onSubmit={this.handleSubmit}>
                <input placeholder="이름" onChange={this.handleChange} 
									value ={this.state.name} name ="name" 
									ref={ref => this.input = ref} />
								// (2) ref를 통해서 해당 input form을 input 변수로 전달한다.
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

## 2. ref의 사용 방법 (2)

위와 같은 방식인데, 문법이 조금 다르다.

```jsx
import React, { Component } from 'react';

class PhoneForm extends Component {

    input = React.createRef();

    state = {
        name : '',
        phone : '',
    }

    handleChange = (e) =>{
        this.setState({
            [e.target.name] : e.target.value
        });
    }

    handleSubmit = (e) =>{

        e.preventDefault();

        this.props.onCreate({
            name: this.state.name,
            phone: this.state.phone
        })

        this.setState({
            name : '',
            phone : '',
        })
				//current라는 키워드 사용한다!
        this.input.current.focus();
    }

    render() {
        return (
            <form onSubmit={this.handleSubmit}>
                <input placeholder= "이름" onChange={this.handleChange}
									value ={this.state.name} name ="name" 
									ref={this.input} />
									//함수 대신 this.input 사용
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

## 3. ref를 사용하는 경우

- Focus
- 특정 DOM의 크기를 가져오는 경우
- 특정 DOM의 스크롤 위치 설정, 스크롤 크기 설정을 해야 하는 경우
- 외부 라이브러리 (D3, Chart.. ) 사용하는 경우
- HTML의 Canvas.. 사용하는 경우
- 그 이외에 DOM에 직접 접근이 꼭 필요한 경우