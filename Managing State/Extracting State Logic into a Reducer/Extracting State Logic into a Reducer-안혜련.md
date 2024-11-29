# state 로직을 reducer로 작성하기

useReducer는 복잡한 state 업데이트 로직을 관리하기 좋은 방법입니다. 

특히, state를 업데이트하는 로직이 여러 이벤트 핸들러로 분산된 경우 reducer 함수를 사용하면 코드의 가독성과 유지보수성이 높아집니다.

//TODO: 이해 필요
### Reducer 함수란?

- state와 action을 받아서 다음 state를 반환하는 함수.
- 컴포넌트 외부에서 state 변경 로직을 관리할 수 있음.
- 순수 함수로 작성되어야 함(같은 입력에 대해 항상 같은 출력 반환).
``` js

function reducer(state, action) {
  switch (action.type) {
    case 'add':
      return [...state, action.payload];
    default:
      throw new Error('Unknown action type');
  }
}
```
### useReducer의 구조

``` js
const [state, dispatch] = useReducer(reducer, initialState);
``` 
- reducer: state 변경 로직이 담긴 함수.
- initialState: 초기 state 값.
- dispatch: action을 reducer에 전달하는 함수.

### Action

- 사용자의 의도를 reducer에 전달하는 객체.
- 일반적으로 type 필드와 추가 데이터(payload)를 포함.
``` js
const action = { type: 'add', payload: { id: 1, text: 'New Task' } };
dispatch(action);
```

## 사용 단계
### 1. 기존 state 로직 분리
- useState와 직접적인 setState 호출을 제거.
- 대신 이벤트 핸들러에서 dispatch로 action 객체를 전달.
``` js

function handleAddTask(text) {
  dispatch({ type: 'added', text });
}
```

### 2. Reducer 함수 작성
- 기존 이벤트 핸들러의 state 업데이트 로직을 reducer 함수로 이동.
``` js

function tasksReducer(state, action) {
  switch (action.type) {
    case 'added':
      return [...state, { id: nextId++, text: action.text, done: false }];
    case 'deleted':
      return state.filter(task => task.id !== action.id);
    default:
      throw new Error('Unknown action type: ' + action.type);
  }
}
```
### 3. useReducer 연결
- 컴포넌트에서 useReducer를 호출하고, reducer 함수와 초기 state를 전달.
```js

const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
```

## 언제 useReducer를 사용할까?
- 상태 업데이트 로직이 복잡하거나 분산되어 있을 때.
- 다수의 이벤트 핸들러가 상태를 관리할 때.
- action의 유형별로 state 변경을 구조적으로 관리하고 싶을 때.

