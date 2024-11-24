#### 하나의 컴포넌트에서 state 업데이트가 여러 이벤트 핸들러로 분산되어 관리가 어려워 진다

#### 문제 해결을 위해 state를 업데이트하는 모든 로직을 reducer를 사용해 컴포넌트 외부의 단일 함수로 통합해 관리할 수 있다.

 

### 학습 내용
- reducer 함수란 무엇인가
- useState에서 useReducer로 리팩토링 하는 방법
- reducer를 언제 사용할 수 있는지
- reducer를 잘 작성하는 방법
 

### reducer를 사용하여 state 로직 통합하기
 
#### 예를 들어 다음의 CUD 기능을 수행하는 각각의 핸들러가 있다고 가정해보자

 
```javascript
function handleAddTask(text) {
    setTasks([...tasks, {
      id: nextId++,
      text: text,
      done: false
    }]);
  }

  function handleChangeTask(task) {
    setTasks(tasks.map(t => {
      if (t.id === task.id) {
        return task;
      } else {
        return t;
      }
    }));
  }

  function handleDeleteTask(taskId) {
    setTasks(
      tasks.filter(t => t.id !== taskId)
    );
  }
 ```

#### 각 이벤트 핸들러는 state를 업데이트하기 위해 setTask를 호출한다.

#### 이처럼 컴포넌트가 커질수록 그 안에서 state를 다루는 로직의 양도 늘어남으로 복잡성은 줄이고 
#### 접근성을 높이기 위해 컴포넌트 내부에 있는 state 로직을 컴포넌트 외부의 "reducer"라고 하는 단일 함수로 옮길 수 있다.

 

### 그 방법은 다음과 같다.

 

### 1. state 를 설정하는 것에서 action 을 dispatch 함수로 전달하는 것으로 바꾸기
 
#### 우선 위에서 작성한 이벤트핸들러의 state 설정 관련 로직을 지워보자

```javascript
function handleAddTask(text) { // 사용자가 "Add"를 누르면 호출
}

function handleChangeTask(task) { // 사용자가 task를 토글하거나 "Save"버튼을 누르면 호출
}

function handleDeleteTask(taskId) { // 사용자가 "Delete"를 누르면 호출
}
 ```

#### reducer를 사용한 state 관리는 useState를 사용하여 state를 직접 설정하는 것과는 다르다.

- useState : state를 설정하여 React에게 "무엇을 할 지를 지시"
- reducer : 이벤트 핸들러에서 "action"을 전달하여 "사용자가 방금한 일" 지정
즉 이벤트 핸들러를 통해 "task를 설정"하는 대신 "task를 /추가/변경/삭제"하는 action을 전달하는 것이다.

 

### 다음의 예시 코드를 보자

```javascript
function handleAddTask(text) {
  dispatch({ // dispatch 함수 안에 넣어준 객체를 "action"이라 한다.
    type: 'added', // 객체의 값으로 일반적으로 어떤 상황이 발생하는지에 대한 최소한의 정보를 담고 있어야 한다
    id: nextId++,
    text: text,
  });
}
```
 

#### 이때 dispatch안의 객체(action)는 어떤 형태든 될 수 있다.

#### 보통 발생한 일을 설명하는 문자열 type을 넘겨주고 정보는 다른 필드에 담아서 전달하는 것이 일반적이다.

 
```javascript
dispatch({
  // 컴포넌트마다 다른 값
  type: 'what_happened',
  // 다른 필드는 이곳에
});
```
 

### 2. reducer 함수 작성하기
#### reducer 함수는 state에 대한 로직을 넣는 곳이다.

#### 다음의 함수는 현재의 state 값과 action 객체를 인자 값으로 받고 다음 state 값을 반환한다.

```javascript
function yourReducer(state, action) {
  // React가 설정하게될 다음 state 값을 반환합니다.
}
```

#### React는 reducer에서 반환된 값을 state에 설정한다.

 

### 먼저 아래의 코드를 살펴보자

 
```javascript
function tasksReducer(tasks, action) {
  if (action.type === 'added') {
    return [...tasks, { // 추가 로직
      id: action.id,
      text: action.text,
      done: false
    }];
  } else if (action.type === 'changed') {
    return tasks.map(t => { // 변경 로직
      if (t.id === action.task.id) {
        return action.task;
      } else {
        return t;
      }
    });
  } else if (action.type === 'deleted') {
    return tasks.filter(t => t.id !== action.id); // 삭제 로직
  } else {
    throw Error('Unknown action: ' + action.type);
  }
}
 
```
#### 1. 첫 번째 인자에 현재 state(tasks) 선언

#### 2. 두 번째 인자에 action 객체 선언

#### 3. reducer에서 다음 state반환(React가 state에 설정하게 될 값)

 

### reducer 함수의 특징

- reducer 함수는 state(tasks)를 인자로 받기 때문에 컴포넌트 외부에서 선언할 수 있다.
- reducer 함수 안에서는 switch문을 사용하는 것이 "규칙"이다.

### 3. 컴포넌트에서 reducer 사용하기
 

#### 마지막으로 컴포넌트에 tasksReducer를 연결할 차례이다.

 
```javascript
import { useReducer } from 'react'; // hook을 불러오고
const [tasks, setTasks] = useState(initialTasks); //선언된 useState를 
const [tasks, dispatch] = useReducer(tasksReducer, initialTasks); //useReducer로 변경
 ```

#### useReducer hook은 두 개의 인자를 넘겨받는다
- reducer 함수
- 초기 state 값
#### 그리고 아래와 같이 반환한다.
- state를 담을 수 있는 값
- dispatch 함수(사용자의 action을 reducer 함수에게 "전달하게 될")
 

### 예시 코드 살펴보기..

 
```javascript
// tasksReducer.js
export default function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [...tasks, {
        id: action.id,
        text: action.text,
        done: false
      }];
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}

// App.js

import { useReducer } from 'react';
import AddTask from './AddTask.js';
import tasksReducer from './tasksReducer.js';

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(
    tasksReducer,
    initialTasks
  );

  function handleAddTask(text) {
    dispatch({
      type: 'added',
      id: nextId++,
      text: text,
    });
  }
 return (
    <>
      <h1>Prague itinerary</h1>
      <AddTask
        onAddTask={handleAddTask}
      />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}

let nextId = 3;
const initialTasks = [
  { id: 0, text: 'Visit Kafka Museum', done: true },
  { id: 1, text: 'Watch a puppet show', done: false },
  { id: 2, text: 'Lennon Wall pic', done: false },
];
 ```

### useState와 useReducer 비교하기
#### 차이는 다음과 같다. 

 

- 코드 크기
    - useState는 코드가 간결하지만, 여러 이벤트 핸들러에서 비슷한 방식으로 상태를 업데이트할 때는 useReducer가 코드 양을 줄일 수 있음.
- 가독성
    - 간단한 상태 업데이트에는 useState가 가독성이 좋지만, 복잡한 상태에서는 useReducer가 업데이트 로직을 명확히 구분해 가독성을 높일 수 있음.
- 디버깅
    - useReducer는 콘솔 로그를 통해 상태 업데이트 과정을 추적하기 용이해 버그를 찾는 데 유리하나, 더 많은 코드 단계에서 디버깅이 필요할 수 있음.
- 테스팅
    - useReducer는 순수 함수로, 독립적으로 테스트하기 용이함. 복잡한 상태 업데이트 로직의 경우 특정 초기 상태와 액션에 대한 테스트가 유용함.
- 개인적인 취향
    - useState와 useReducer는 서로 대체 가능하며, 사용자의 선호에 따라 선택할 수 있음.
 

### reducer를 작성할 때 다음의 두가지 팁을 생각하자
 

- Reducer는 반드시 순수해야 한다. 
    - 즉 객체나 배열을 변경하지 않고(map이나 filter 또는 복사를 통해)업데이트 해야한다.
- 각 action(객체)은 데이터 안에서 여러 변경들이 있더라도 하나의 사용자 상호 작용을 설명해야한다.
    - 사용자가 reducer가 관리하는 5개의 필드가 있는 양식에서 '재설정'을 누른 경우, 5개의 개별 set_field action보다는 하나의 reset_form action을 전송하는 것이 더 합리적 이다)
 

### Immer로 간결한 reducer 작성하기
#### 아래와 같이 useImmerReducer를 사용하여 push 또는 arr[i] = 로 값을 할당하여 state를 변경할 수 있다.
```javascript
    case 'changed': {
      const index = draft.findIndex(t =>
        t.id === action.task.id
      );
      draft[index] = action.task;
      break;
    }
```