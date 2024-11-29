컴포넌트가 복잡해지면 state가 업데이트 되는 다양한 경우를 파악하기 어려워짐 
-> 컴포넌트 내부의 state를 업데이트하는 모든 로직을 reducer라는 단일 함수를 사용해 컴포넌트 외부로 옮겨 관리 가능(reducer는 컴포넌트에 의존하지 않는 순수함수임-렌더링 중에 실행되니까)

### state를 reducer로 관리하도록 리팩토링하는 방법

1. state를 설정하는 것 대신 `action객체`로 발생하는 일과, 추가 데이터를 정의하고
action을 `dispatch함수`로 reducer함수에 전달함

```jsx
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
// 대신
function handleAddTask(text) {
  dispatch({
    type: 'added', //action 객체의 type 필드에 발생하는 일을 설명하는 문자열을 넘김
    id: nextId++,
    text: text,
  });
}

function handleChangeTask(task) {
  dispatch({
    type: 'changed',
    task: task
  });
}

function handleDeleteTask(taskId) {
  dispatch({
    type: 'deleted',
    id: taskId
  });
}
```

2. `reducer함수` 작성하기

reducer 함수는 현재state값, action객체, 이렇게 두 개의 인자를 받고 다음 state 값을 반환함 (state를 인자로 받으므로 이를 컴포넌트 외부에 선언 가능->코드의 가독성 up)

React는 reducer가 반환한 값을 state에 설정함 

-> 이벤트 핸들러는 action을 전달해 무슨 일이 일어나는지만 명시. reducer 함수가 이에 대한 응답으로 state가 어떤값으로 업데이트 될지 결정하므로. 관심사가 분리됨


```jsx
function tasksReducer(tasks, action) {
  switch (action.type) { //reducer에는 일반적으로 switch문 씀
    case 'added': {
      return [...tasks, {
        id: action.id,
        text: action.text,
        done: false
      }];
    }
    case 'changed': {
      return tasks.map(t => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter(t => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}
```

⚠️ reducer 사용시 주의점

- **각 action은 하나의 상호작용을 설명해야함**
e.g. 사용자가 reducer가 관리하는 3개의 필드가 있는 양식에서 '재설정'을 누르면 3개의 개별 set_filed가 아닌 하나의 reset_form action을 전송해야함

- **reducer는 순수함수니까 컴포넌트 외부에 영향을 미치는 작업을 수행해선 안됨 / 객체,배열의 불변성을 유지해야함**
객체, 배열을 다루는 경우 Immer 라이브러리에서 제공하는 `useImmerReducer`를 사용해 간결하게 코드 작성도 가능
```jsx
function tasksReducer(draft, action) {
  switch (action.type) {
    case 'added': {
      draft.push({
        id: action.id,
        text: action.text,
        done: false
      });
      break;
    }
    case 'changed': {
      const index = draft.findIndex(t =>
        t.id === action.task.id
      );
      if(index) draft[index] = action.task;
      break;
    }
    case 'deleted': {
      return draft.filter(t => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}

export default function TaskApp() {
  const [tasks, dispatch] = useImmerReducer(
    tasksReducer,
    initialTasks
  );
  ...
}
```

3. 컴포넌트에서 reducer 사용하기

`useReducer`훅으로 컴포넌트에 reducer 함수 연결

`useReducer`훅은 state를 설정하는 함수(reducer), 초기 state값을 입력받아
state를 담을 수 있는 값과 dispatch함수를 리턴함

```jsx
import { useReducer } from 'react';
const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

function handleAddTask(text) {
  dispatch({
    type: 'added',
    id: nextId++,
    text: text,
  });
}

function handleChangeTask(task) {
  dispatch({
    type: 'changed',
    task: task
  });
}

...
```

### useState vs useReducer 

||useState|useReducer|
|----|------------|------------|
|코드 크기|미리 작성하는 코드는 적음|reducer함수, action을 전달하는 부분을 다 작성해야한다는 단점 but **`여러 이벤트 핸들러에서 비슷한 방식으로 state를 업데이트할 땐`** 코드 양 줄여줌|
|가독성|간단한 state 업데이트에선 가독성 좋음|**`복잡한 구조의 state를 다룰 땐`** reducer함수로 업데이트 로직, 이벤트핸들러로 무엇이 발생했는지 명확히 구분 가능|
|디버깅|디버깅 어려움|콘솔 로그를 각 단계에 추가해 state가 업데이트 중 어느 부분에서 버그가 발생했는지 확인 용이|

동일한 방식이기 때문에
같은 컴포넌트에서 useState와 useReducer를 섞어 써도 되고, 언제나 바꿀 수 있음



