# React : JSX 문법

분류: React
작성일시: 2021년 7월 8일 오후 12:16

## 1. JSX

- 리액트 컴포넌트를 작성할 때 사용하는 문법,
- HTML과 비슷하지만, Javascript로 변환되어 작성됨
- 참고문서 : [https://react-anyone.vlpt.us/03.html](https://react-anyone.vlpt.us/03.html)

- JSX 문법
    1. 태그는 꼭 닫혀있어야 한다 ( 만약 <div>로 열었다면, </div>로 닫아줘야 한다)

        ```jsx
        import React, { Component } from 'react';

        class App extends Component {
          render() {
            return (
              <div>
                <input type="text"> // (x)
                <input type="text" /> // (o) : Self-closing tag
                <input type="text"> </input> // (o)
              </div>
            );
          }
        }

        export default App;
        ```

    2. 두 개 이상의 엘리먼트는 무조건 하나의 엘리먼트로 감싸져 있어야 한다.

        ```jsx
        import React, { Component } from 'react';

        class App extends Component {
          render() {
            return (
              <div>
                Hello
              </div>
              <div>
                Bye
              </div>  // (x), div가 감싸져 있지 않음

        		<div>
        			<div>
                Hello
              </div>
              <div>
                Bye
              </div>  
        		</div>   // (o)
            );
          }
        }

        export default App;
        ```

        하지만 이런 경우 불필요한 <div> 가 추가된다.

        이러한 점을 수정하기 위해 Fragment ( [https://reactjs.org/docs/fragments.html](https://reactjs.org/docs/fragments.html) ) 를 사용할 수 있다. 

        ```jsx
        import React, { Component, Fragment } from 'react';

        class App extends Component {
          render() {
            return (
              <Fragment>
                <div>
                  Hello
                </div>
                <div>
                  Bye
                </div>
              </Fragment>
            );
          }
        }
        ```

        이런 경우, 실제 render 될 때 불필요한 div 가 없어짐을 확인할 수 있다.

    3. { } 을 통해 JSX 내부에서 자바스크립트 값을 사용할 수 있다.

        ```jsx
        import React, { Component } from 'react';

        class App extends Component {
          render() {
            const name = 'react';
            return (
              <div>
                hello {name}! // hello, react!
              </div>
            );
          }
        }

        export default App;
        ```

    4. 조건부 렌더링 : If문 사용 불가, 사용하려면 [IIFE](https://developer.mozilla.org/ko/docs/Glossary/IIFE) (즉시 실행 함수 표현) 사용해야 함

    따라서 삼항 연산자, AND 연산자 주로 이용

    - 삼항 연산자, (맞아요! 출력),

        TRUE, FALSE일때 각각 다른 값 보여주고 싶을때 출력

    ```jsx
    import React, { Component } from 'react';

    class App extends Component {
      render() {
        return (
          <div>
            {
              1 + 1 === 2 
                ? (<div>맞아요!</div>)
                : (<div>틀려요!</div>)
            }
          </div>
        );
      }
    }

    export default App;
    ```

    - AND 연산자

        ( 값이 TRUE일때만 출력하도록 하는 경우)

    ```jsx
    import React, { Component } from 'react';

    class App extends Component {
      render() {
        return (
          <div>
            {
              1 + 1 === 2 && (<div>맞아요!</div>)
            }
          </div>
        );
      }
    }

    export default App;
    ```

    - IIFE ( 보통 JSX 밖에서 로직을 작성하는게 좋지만, 꼭 JSX 내부에서 작성해야 한다면)

    ```jsx
    import React, { Component } from 'react';

    class App extends Component {
      render() {
        const value = 1; //값 바뀌면 render도 바뀜
        return (
          <div>
            {
              (function() {
                if (value === 1) return (<div>하나</div>);
                if (value === 2) return (<div>둘</div>);
                if (value === 3) return (<div>셋</div>);
              })()

    					(() => {
    					  if (value === 1) return (<div>하나</div>);
    					  if (value === 2) return (<div>둘</div>);
    					  if (value === 3) return (<div>셋</div>);
    					})()
            }
          </div>
        );
      }
    }

    export default App;
    ```

    위와 아래 같은 문, 밑의 경우는 [화살표 함수](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Functions/Arrow_functions) 라고 부름

    5. style과 className

    - Style은 객체 형태로 작성

    ```jsx
    import React, { Component } from 'react';

    class App extends Component {
      render() {
        const style = {
          backgroundColor: 'black',
          padding: '16px',
          color: 'white',
          fontSize: '36px'
        };

        return <div style={style}>안녕하세요!</div>;
      }
    }

    export default App;
    ```

    - HTML에서의 class는 className으로 사용

    - App.css

    ```css
    .App {
      background: black;
      color: aqua;
      font-size: 36px;
      padding: 1rem;
      font-weight: 600;
    }
    ```

    - App.js

    ```jsx
    import React, { Component } from 'react';
    import './App.css'

    class App extends Component {
      render() {
        return (
          <div className="App">
            리액트
          </div>
        );
      }
    }

    export default App;
    ```

    6. 주석

    ```jsx
    import React, { Component } from 'react';

    class App extends Component {
      render() {
        return (
          <div>
            {/* 주석은 이렇게 */}
            <h1
              // 태그 사이에
            >리액트</h1>
          </div>
        );
      }
    }

    export default App;
    ```

### TIP : var, let, const

- var : 함수 Scope (단, ES6 이후부터는 사용하지 않음)
- let : 블록 Scope
- const : 바뀌지 않을 값 (블록 scope)