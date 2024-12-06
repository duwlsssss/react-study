# state 로직을 reducer로 작성하기

한 컴포넌트에서 state 업데이트가 여러 이벤트 핸들러로 분산되는 경우가 있습니다. 이 경우 컴포넌트를 관리하기 어려워집니다. 이 문제는 state를 업데이트하는 모든 로직을 reducer를 사용해 컴포넌트 외부의 단일 함수로 통합해 관리할 수 있습니다.

## reducer를 사용하여 state 로직 통합하기

컴포넌트가 복잡해지면 컴포넌트의 state가 업데이트되는 다야한 경우를 한눈에 파악하기 어려워질 수 있습니다.
컴포넌트가 커질수록 그 안에서 state를 다루는 로직의 양도 늘어나게 됩니다. 복잡성은 줄이고 접근성을 높이기 위해서, 컴포넌트 내부에 있는 state로직을 컴포넌트 외부의 "reducer"라는 단일 함수로 옮길 수 있습니다.

reducer는 state를 다루는 다른 방법입니다. 다음과 같은 세가지 단계에 걸쳐 `useState`에서 `useReducer`로 바꿀 수 있습니다.

### 1. state를 설정하는 것에서 action을 dispatch 함수로 전달하는 것으로 바꾸기
reducer를 사용한 state 관리는 state를 직접 설정하는 것과 약간 다릅니다. state를 설정하여 React에게 "무엇을 할 지"를 지시하는 대신, 이벤트 핸들러에서 "Action"을 전달하여 "사용자가 방금 한 일"을 지정합니다. 즉, 이벤트 핸들러를 통해 "`task`를 설정"하는 대신 "task를 추가/변경/삭제"하는 action을 전달하는 것입니다. 이러한 방식이 사용자의 의도를 더 명확하게 설명합니다.
```jsx
function handleDeleteTask(taskId) {
  dispatch(
    // "action" 객체:
    {
      type: 'deleted',
      id: taskId
    }
  );
}

//예시 
function handleAddTask(text) {
  setTasks([...tasks, {
    id: nextId++,
    text: text,
    done: false
  }]);
}
//아래로 변경
function handleAddTask(text) {
  dispatch({
    type: 'added',
    id: nextId++,
    text: text,
  });
}
```
`dispatch`함수에 넣어준 객체를 "action"이라고 합니다.

이 객체는 일반적인 자바스크립트 객체입니다. 이 안에 어떤 것이든 자유롭게 넣을 수 있지만 일반적으로 어떤 상황이 발생하는지에 대한 최소한의 정보를 담고 있어야합니다.

action객체는 어떤 형태든 될 수 있습니다. 하지만 발생한 일을 설명하는 문자열 `type`을 넘겨주고 이외의 정보는 다른 필드에 담아서 전달하도록 작성하는게 일반적입니다. `type`은 컴포넌트에 따라 값이 다르며 이 예시의 경우 `added` 또는 `added_task` 둘 다 괜찮습니다. 무슨 일이 일어나는지 설명할 수 있는 이름을 넣어주면 됩니다.
```jsx
dispatch({
  // 컴포넌트마다 다른 값
  type: 'what_happened',
  // 다른 필드는 이곳에
});
```

### 2. reducer 함수 작성하기
reducer 함수는 state에 대한 로직을 넣는 곳입니다. 이 함수는 현재의 state 값과 action 객체, 이렇게 두개의 인자를 받고 다음 state값을 반환합니다.
```jsx
function yourReducer(state, action) {
  // React가 설정하게될 다음 state 값을 반환합니다.
}
```
React는 reducer에서 반환한 값을 state에 설정합니다.
이 예시에서 이벤트 핸들러에 구현 되어 있는 state설정과 관련 로직을 reducer 함수로 옮기기 위해서 다음과 같이 해보겠습니다.

1. 첫 번째 인자에 state(`tasks`)선언하기
2. 두 번째 인자에 `action`객체 선언하기
3. reducer에서 다음 state반환하기(React가 state에 설정하게 되는 값)

```jsx
function tasksReducer(tasks, action) {
  switch (action.type) {
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
reducer함수는 state를 인자로 받고 있기 때문에, 이를 컴포넌트 외부에서 선언할 수 있습니다. 이 행동으로 들여쓰기 수준이 줄어들고 가독성을 높일 수 있습니다.

reducer 함수 안에서는 switch문을 사용하는 것이 규칙입니다. 결과는 if/else문과 같지만, switch문으로 작성하는 것이 한눈에 읽기 더 쉬울 수 있습니다.
각자 다른 `case`속에서 선언된 변수들이 서로 충돌하지 않도록 `case`블록을 중괄호`{}`로 감싸는 것이 좋습니다. 또한 일반적인 `case`는 `return`으로 끝나야합니다. `return`하는 것을 잊으면 다음 `case`로 떨어져 실수할 수 있습니다.

### 3. 컴포넌트에서 reducer 사용하기

```jsx
import{ useReducer } from 'react';
const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
```

reducer를 사용하기 위해서는 `useReducer` hook을 불러와 사용해야합니다. `useReducer` hook은 초기 sate 값을 입력받아 stateful한 값을 반환하는 점과 state를 설정하는 함수의 원리를 보면 `useState`와 비슷합니다. 하지만 `useReducer`는 조금 다른점이 있습니다.

`useReducer`는 reducer 함수와 초기 state 값 이 두 가지 인자를 넘겨 받습니다.
그리도 'state를 담을 수 있는 값'과 dispatch함수(사용자의 action을 reducer 함수에게 전달)를 반환합니다.

```jsx
import { useReducer } from 'react';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';

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
// taskReducer를 다른 파일로 분리하는 것도 가능!
function tasksReducer(tasks, action) {
  switch (action.type) {
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

let nextId = 3;
const initialTasks = [
  { id: 0, text: 'Visit Kafka Museum', done: true },
  { id: 1, text: 'Watch a puppet show', done: false },
  { id: 2, text: 'Lennon Wall pic', done: false }
];
```
위 예제는 이제 `useState`가 아닌 `useReducer`로 state를 관리합니다.

### `useState`와 `useReducer` 비교하기

reducer도 단점은 있습니다. 이제부터 `useState`와 `useReducer`를 비교할 수 있는 몇 가지 방법을 알아보겠습니다

- 코드 크기: `useState`를 사용하면 미리 작성해야 하는 코드가 줄어듭니다. 하지만 `useReducer`는 reducer함수 그리고 action을 전달하는 부분 둘 다 작성해야 합니다. 단 여러 이벤트 핸들러에서 비슷한 방식으로 state를 업데이트 하는 경우, `useReducer`를 사용하면 코드의 양을 줄이는데 도움이 될 수 있습니다.
- 가독성: `useState`로 간단한 state를 업데이트하는 경우 가독성이 좋은 편입니다. 하지만 더 복잡한 구조의 state를 다루게 되면 컴포넌트의 코드 양이 더 많아져 한눈에 읽기 어려워질 수 있습니다. 이 경우 `useReducer`를 사용하면 업데이트 로직이 어떻게 동작하는지, 이벤트 핸들러를 통해서 무엇이 발생했는지 구현한 부분을 명확하게 구분할 수 있습니다.
- 디버깅: `useState`를 사용하며 버그를 발견했을 때, 문제가 어디서 발생했는지 찾기 어려울 수 있습니다. `useReducer`를 사용하면 콘솔 로그를 reducer에 추가해 state가 업데이트 되는 모든 부분과 왜 버그가 발생했는지, 어떤 `action`으로 인한 것인지 확인 할 수 있습니다. 하지만 `useState`를 사용하는 경우보다 더 많은 코드를 실행해서 단계적으로 디버깅해야 하는 점이 있기도 합니다.
- 테스팅: reducer는 컴포넌트에 의존하지 않는 순수 함수입니다. 따라서 독립적으로 분리해서 내보내거나 테스트 할 수 있습니다. 일반적으로 더 현실적인 환경에서 컴포넌트 테스트를 하는것이 좋지만, 복잡한 state를 업데이트 하는 로직의 경우 reducer가 특정 초기 state 및 action에 대해 특정 state를 반환 한다고 생각하고 테스트하는 것이 유용할 수 있습니다.

## reducer 잘 작성하기

- Reducer는 반드시 순수해야합니다. reducer는 `setState`와 비슷하게 랜더링 중에 실행됩니다.(action은 다음 랜더링까지 대기) 즉 reducer는 순수해야 한다는 것입니다.
- 각 action은 데이터 안에서 여러 변경들이 있더라도 하나의 사용자 상호작용을 설명해야합니다. 예를 들어 reducer가 관리하는 5개의 필드가 있는 양식에서 재설정을 누른 경우 5개의 개별 action보다는 하나의 action을 전송하는 것이 합리적입니다. 모든 action을 기록하면 어떤 상호작용이나 응답이 어떤 순서로 일어났는지 재구성할 수 있을 만큼 로그가 명확해야 합니다. 이는 디버깅에 도움이 되죠!
