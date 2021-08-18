# 생활코딩 : Redux

분류: REDUX
자료 링크: https://www.youtube.com/watch?v=Jr9i3Lgb5Qc&list=PLuHgQVnccGMB-iGMgONoRPArZfjRuRNVc&index=1
작성일시: 2021년 8월 18일 오후 12:52

## 1. 리덕스의 전체 구성도

![Untitled](https://github.com/LemonDouble/TIL/blob/main/Redux/img/Untitled.png)

- State : State 저장소
- Reducer : 유저가 작성하는 함수, State에 저장하는 로직을 구현한다.
- Render : UI를 만들어주는 코드
- getState : state의 getter
- subscribe : Render 함수가 등록되면, State가 바뀔 때마다 Render 함수를 호출해준다.

- Action : 유저가 예를 들어 글을 작성하고 Submit 버튼을 누르는 등, 이벤트가 발생하였을 때 dispatch로 전달되는 객체, type : 'create', payload : {title : title, desc:desc} 와 같이, 필수적으로 Type을 가지고, 그 외 프로퍼티는 유저가 설정할 수 있다.
- dispatch : Action을 reducer로 전달한다. 이후 Reducer는 State 변경 후 새로운 State를 subscribe에 전달하고, subscribe는 render에게 값을 전달하고, render 함수는 UI를 변경한다.

## 2. Why Redux?

![Untitled](https://github.com/LemonDouble/TIL/blob/main/Redux/img/Untitled%201.png)

- Redux가 없는 경우
    - Component가 추가될 때마다 상태에 관련된 로직을 새로 작성해 주어야 한다.
    - 따라서, Component가 늘어날 때마다 작성해야 하는 코드와 상태가 기하급수적으로 늘어난다.
    - 또한, 각 Component가 강하게 결합되게 되므로 유지보수가 어려워진다.

![Untitled](https://github.com/LemonDouble/TIL/blob/main/Redux/img/Untitled%202.png)

- Redux가 있는 경우
    - Redux State Storage와 Component가 결합함으로써 각 Component는
        - 자신이 State를 변경하는 로직
        - State가 변경되었을 때 행동할 로직
    - 두 가지만을 고려하면 된다.
    - 따라서, 아무리 Component 개수가 늘어나도 2 * Component 개수만큼의 로직만 작성하면 된다!

- 또한 Redux를 사용하는 경우, 각 State의 Snapshot(Vesion)을 관리할 수 있다.
    - 따라서 쉽게 Redo, Undo가 가능하다

## 3. Redux 없이 어플리케이션 만들기

- 컴포넌트가 증가할때마다, 주석 처리된 아래 로직들을 전부 수정/변경해줘야 한다.

```jsx
<html>
  <body>
    <style>
      .container {
        border: 5px solid black;
        padding: 10px;
        margin: 10px;
      }
    </style>
    <div id="red"></div>
    <div id="green"></div>
    <div id="blue"></div>
    <script>
      function red() {
        document.querySelector("#red").innerHTML = `
                <div class="container" id="component_red">
                    <h1> red </h1>
                    <input type="button" value = "fire"
                    onclick = "
                    <!-- 컴포넌트가 추가될 때마다 이 로직들을 전부 수정해줘야 한다! --> 
                    document.querySelector('#component_red').style.backgroundColor = 'red';
                    document.querySelector('#component_green').style.backgroundColor = 'red';
                    document.querySelector('#component_blue').style.backgroundColor = 'red';
                    ">
                </div>
            `;
      }
      function green() {
        document.querySelector("#green").innerHTML = `
                <div class="container" id="component_green">
                    <h1> green </h1>
                    <input type="button" value = "fire"
                    onclick = "
                    document.querySelector('#component_red').style.backgroundColor = 'green';
                    document.querySelector('#component_green').style.backgroundColor = 'green';
                    document.querySelector('#component_blue').style.backgroundColor = 'green';
                    ">
                </div>
            `;
      }
      function blue() {
        document.querySelector("#blue").innerHTML = `
                <div class="container" id="component_blue">
                    <h1> blue </h1>
                    <input type="button" value = "fire"
                    onclick = "
                    document.querySelector('#component_red').style.backgroundColor = 'blue';
                    document.querySelector('#component_green').style.backgroundColor = 'blue'
                    document.querySelector('#component_blue').style.backgroundColor = 'blue';
                    ">
                </div>
            `;
      }
      red();
      green();
      blue();
    </script>
  </body>
</html>
```

## 4. Redux를 통해 다시 만들기

```html
<html>
  <head>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/redux/4.1.1/redux.js"></script>
  </head>
  <body>
    <style>
      .container {
        border: 5px solid black;
        padding: 10px;
        margin: 10px;
      }
    </style>
    <div id="red"></div>
    <div id="blue"></div>
    <div id="green"></div>
    <script>
      function reducer(state, action) {
        console.log(state, action);
        //state가 undefined라면, State를 변경하는 것이 아닌 최초 초기화 단계
        //따라서 아래는 State의 초기 값을 설정한다.
        //state : 이전의 state, action : 전달된 action
        if (state === undefined) {
          return { color: "yellow" };
        }
        /*
        Error :
        state를 직접 수정하지 말고, state를 복사한 후 복사한 State를 수정하자.
        불변성(Immutable) 을 유지해줘야 한다.
        if (action.type === "CHANGE_COLOR") {
          state.color = "red";
        }
        */

        //Immutable을 유지하면서 State 수정
        var newState;
        if (action.type === "CHANGE_COLOR") {
          //이전 State를 복사한다.
          //이후 전달받은 Action의 State로 변경시킨다.
          newState = Object.assign({}, state, { color: action.color });
        }

        return newState;
      }

      //window 부분은 Redux dev tools를 사용하기 위해 추가된 코드
      //Redux dev tools 사용하기 위해서는 서버 환경임이 필요하다.
      //만약 로컬에서 개발한다면 Vscode의 Live Server등 사용하자.
      //Redux dev tools Extension 설치하여 Chrome F12 -> Redux 탭 누르면 State 시간여행이 가능하다.
      //State를 Download, Upload 하는 기능도 있다.

      const store = Redux.createStore(
        reducer,
        window.__REDUX_DEVTOOLS_EXTENSION__ &&
          window.__REDUX_DEVTOOLS_EXTENSION__()
      );

      //초기값 확인, {color: "yellow"}
      console.log(store.getState());

      //red 함수는 현재 state를 받아와 바꿔주므로, 언제든지 호출해도 되는 함수이다. (render)
      //따라서, Subscribe에 red 함수를 등록하면 된다.
      function red() {
        //state를 받아오고, 초기값으로 yellow를 render한다.
        var state = store.getState();
        document.querySelector("#red").innerHTML = `
                <div class="container" id="component_red" style="background-color:${state.color}">
                    <h1> red </h1>
                    <input type="button" value = "fire"
                    onclick = "
                       store.dispatch({type:'CHANGE_COLOR', color: 'red'});
                    ">
                </div>
            `;
      }

      //State가 바뀔 때마다 red 함수가 호출되도록 등록
      store.subscribe(red);
      red();

      // 아래로, 컴포넌트를 추가하더라도 상태가 바뀜을 Store에 Dispatch해주고,
      // State가 바뀔 때마다 render 함수만 작성해주면 된다.
      // red는 blue, green을 모르고, blue는 red와 green을 모른다.
      // 즉, 각 Component는 자신의 일에만 집중하면 된다.

      function blue() {
        var state = store.getState();
        document.querySelector("#blue").innerHTML = `
                <div class="container" id="component_blue" style="background-color:${state.color}">
                    <h1> blue </h1>
                    <input type="button" value = "fire"
                    onclick = "
                       store.dispatch({type:'CHANGE_COLOR', color: 'blue'});
                    ">
                </div>
            `;
      }
      store.subscribe(blue);
      blue();

      function green() {
        var state = store.getState();
        document.querySelector("#green").innerHTML = `
                <div class="container" id="component_green" style="background-color:${state.color}">
                    <h1> green </h1>
                    <input type="button" value = "fire"
                    onclick = "
                       store.dispatch({type:'CHANGE_COLOR', color: 'green'});
                    ">
                </div>
            `;
      }
      store.subscribe(green);
      green();
    </script>
  </body>
</html>
```

## 5. Redux를 사용한 CRUD 구현

- 시간관계상 Update는 빠져있다 머쓱;;

```html
<!DOCTYPE html>
<html>
  <head>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/redux/4.1.1/redux.js"></script>
  </head>
  <body>
    <div id="subject"></div>
    <div id="toc"></div>
    <div id="control"></div>
    <div id="content"></div>
    <script>
      //이렇게 작성하는 이유 : 모듈화
      //가독성과 재사용성을 고려할 수 있다.
      function subject() {
        document.querySelector("#subject").innerHTML = `
        <header>
          <h1>WEB</h1>
          Hello, WEB!
        </header>
        `;
      }

      function TOC() {
        const state = store.getState();
        let i = 0;
        let liTags = "";
        while (i < state.contents.length) {
          liTags += `
          <li>
              <a onclick ="
                event.preventDefault();
                let action = {type:'SELECT', 
                id :${state.contents[i].id},
                mode : 'read'
              }
                store.dispatch(action);
              "
              href="${state.contents[i].id}">${state.contents[i].title}</a>
          </li>
          `;
          i = i + 1;
        }
        document.querySelector("#toc").innerHTML = `
        <nav>
          <ol>
            ${liTags}
          </ol>
        </nav>
        `;
      }

      function control() {
        document.querySelector("#control").innerHTML = `
        <ul>
          <li>
            <a onclick ="
              event.preventDefault();
              store.dispatch({type:'CHANGE_MODE_CREATE'});
            "
            href="/create">create</a>
          </li>
          <li><input onclick="
              store.dispatch({
                type : 'DELETE'
              })
            "
            type="button" value="delete" /></li>
        </ul>
        `;
      }

      function article() {
        const state = store.getState();
        if (state.mode === "create") {
          document.querySelector("#content").innerHTML = `
            <article>
              <form onsubmit ="
                event.preventDefault();
                let _title = this.title.value;
                let _desc = this.desc.value;
                store.dispatch({
                  type:'CREATE',
                  title:_title,
                  desc: _desc,
                })
              ">
              <form>
                <p>
                  <input type="text" name ="title" placeholder ="title">
                </p>
                <p>
                  <textarea name="desc" placeholder="description"></textarea>
                </p>
                <p>
                  <input type="submit">
                </p>
              </form>
            </article>
            `;
        } else if (state.mode === "read") {
          let i = 0;
          let aTitle, aDesc;
          while (i < state.contents.length) {
            if (state.contents[i].id === state.selected_id) {
              aTitle = state.contents[i].title;
              aDesc = state.contents[i].desc;
              break;
            }
            i = i + 1;
          }
          document.querySelector("#content").innerHTML = `
          <article>
            <h2>${aTitle}</h2>
            ${aDesc}
          </article>
          `;
        } else if (state.mode === "welcome") {
          document.querySelector("#content").innerHTML = `
          <article>
            <h2>Welcome</h2>
            Hello, Redux!!
          </article>
          `;
        }
      }

      function reducer(state, action) {
        console.log("reducer called!");
        if (state === undefined) {
          return {
            max_id: 2,
            mode: "welcome",
            selected_id: null,
            contents: [
              { id: 1, title: "HTML", desc: "HTML is ..." },
              { id: 2, title: "CSS", desc: "CSS is ..." },
            ],
          };
        }

        let newState;
        if (action.type === "SELECT") {
          newState = Object.assign({}, state, {
            selected_id: action.id,
            mode: action.mode,
          });
        }

        if (action.type === "CHANGE_MODE_CREATE") {
          newState = Object.assign({}, state, { mode: "create" });
        }

        if (action.type === "CREATE") {
          let newContents = state.contents.concat();
          let new_max_id = state.max_id + 1;
          newContents.push({
            id: new_max_id,
            title: action.title,
            desc: action.desc,
          });

          newState = Object.assign({}, state, {
            max_id: new_max_id,
            contents: newContents,
            selected_id: new_max_id,
            mode: "read",
          });
        }

        if (action.type === "DELETE") {
          let newContents = [];
          let i = 0;
          while (i < state.contents.length) {
            if (state.selected_id !== state.contents[i].id) {
              newContents.push(state.contents[i]);
            }
            i = i + 1;
          }

          newState = Object.assign({}, state, {
            contents: newContents,
            mode: "welcome",
          });
        }
        console.log(action, state, newState);
        return newState;
      }

      const store = Redux.createStore(reducer);

      subject();
      TOC();
      store.subscribe(TOC);
      control();
      article();
      store.subscribe(article);
    </script>
  </body>
</html>
```
