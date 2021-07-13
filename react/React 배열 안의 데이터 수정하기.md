# React : 배열 안의 데이터 수정하기

분류: React
작성일시: 2021년 7월 13일 오후 7:29

## 1. Slice, Map

- Slice

```jsx
const numbers = [1,2,3,4,5];

[
	...numbers.slice(0,2),
	9,
	...numbers.slice(3,5)
]
// [1,2,9,4,5]
```

- Map

```jsx
numbers.map(n => {if (n == 3) {return 9;}else return n})
//[1,2,9,4,5]
```

## 2. Update 구현

1. App.js에서 Update 핸들러를 구현한다.

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

  handleRemove = (id) =>{
    const {information} = this.state;

    this.setState({
      information : information.filter(info => info.id !== id)
    });
  }

  handleUpdate = (id,data) => {
    const {information} = this.state;
    this.setState({
      information: information.map(
        info => {
          if(info.id === id){
            return{
              id,
              ...data,
            }
          }
          return info;
        }
      )
    })
  }
  
  render() {
    return (
      <div>
        <PhoneForm onCreate={this.handleCrate}/>
        <PhoneInfoList 
          data={this.state.information}
          onRemove={this.handleRemove}
					{//onUpdate 핸들러를 넘긴다}
          onUpdate={this.handleUpdate}
        />
      </div>
    );
  }
}

export default App;
```

이때, Update는 (1)의 Map을 사용하여 id가 같다면 수정하고, 아니라면 원래 데이터를 그대로 사용한다.

2. PhoneInfoList.js를 수정하여 PhoneList에게 onUpdate 핸들러를 넘긴다.

```jsx
import React, { Component } from 'react';
import PhoneInfo from './PhoneInfo';

class PhoneInfoList extends Component {

    static defaultProps = {
        data: []
    }
    
    render() {
				//onUpdate 핸들러를 넘긴다.
        const {data, onRemove, onUpdate} = this.props;
        const list = data.map(
            info => (<PhoneInfo onRemove={onRemove} onUpdate={onUpdate} info={info} key={info.id} />)
        );
        return (
            <div>
                {list}
            </div>
        );
    }
}

export default PhoneInfoList;
```

3. PhoneInfo.js를 수정한다.

```jsx
import React, { Component, Fragment} from 'react';

class PhoneInfo extends Component {

		//editing : 수정 모드
    state = {
        editing : false,
        name : '',
        phone : '',
    }

    handleRemove = () =>{
        const {info, onRemove} = this.props;
        onRemove(info.id);
    }

    handleToggleEdit = () =>{

        const {info, onUpdate} = this.props;

        //true -> false 경우
				//onUpdate 핸들러를 통해 Update 수행한다. (수정 적용)
        if(this.state.editing){ 
            onUpdate(info.id,{
                name : this.state.name,
                phone : this.state.phone
            });
        } else{//false -> true 경우
            //state에 info값들 넣어주기
						//input 값에 
            this.setState({
                name : info.name,
                phone : info.phone,
            })
        }
        
        this.setState({
            editing : !this.state.editing,
        })
    }

		//수정 모드일 때, onChange를 걸어 줘 State를 바꿔 준다.
    handleChange = (e) =>{
        this.setState({
            [e.target.name] : e.target.value
        })
    }

    render() {
        const {name, phone} = this.props.info;
        const {editing} = this.state;

        const style ={
            border : '1px solid black',
            padding : '8px',
            margin:'8px',
        };

        return (
            <div style={style}>
                {
                    editing ?(
                        //Editing값이 True면 보여줄 내용
                        <Fragment>
                            <div><input onChange={this.handleChange} value={this.state.name} name="name"/></div>
                            <div><input onChange={this.handleChange} value={this.state.phone} name = "phone"/></div>
                        </Fragment>
                    ) :(
												//Editing값이 False면 보여줄 내용
                        <Fragment>
                        <div><b>{name}</b></div>
                        <div>{phone}</div>
                        </Fragment>
                    )
                }
                <button onClick={this.handleToggleEdit}>{editing ? '적용' : '수정'}</button>
                <button onClick={this.handleRemove}>삭제</button>
            </div>
        );
    }
}

export default PhoneInfo;
```