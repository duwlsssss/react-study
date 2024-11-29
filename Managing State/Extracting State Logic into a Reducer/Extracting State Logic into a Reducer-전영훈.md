# State 로직을 reducer로 작성하기

##reducer란

reducer는 state를 업데이트하는 모든 로직을 컴포넌트 외부의 단일 함수로 통합해 관리하는 도구입니다.

reducer로 state를 관리하는 법

### state를 설정하는 것에서 action을 dispatch 함수로 전달하는 것으로 바꾸기.

State는 react에게 무엇을 할지 지시하는 대신, reducer는 이벤트 핸들러에서 action을 전달하여 사용자가 방금 한 일을 지정합니다.
=> 이벤트 핸들러를 통해 state의 action을 전달하는 것입니다.

```jsx
function handleAddTask(text) {
  dispatch({
    type: "added",
    id: nextId++,
    text: text,
  });
}

function handleChangeTask(task) {
  dispatch({
    type: "changed",
    task: task,
  });
}

function handleDeleteTask(taskId) {
  dispatch({
    type: "deleted",
    id: taskId,
  });
}
```

dispatch 함수에 있는 객체가 action입니다.

### reducer 함수 작성하기.

Reducer 함수는 현재의 state 값과 action 객체, 이렇게 두 개의 인자를 받고 다음 state 값을 반환합니다.

```jsx
function tasksReducer(tasks, action) {
  switch (action.type) {
    case "added": {
      return [
        ...tasks,
        {
          id: action.id,
          text: action.text,
          done: false,
        },
      ];
    }
    case "changed": {
      return tasks.map((t) => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case "deleted": {
      return tasks.filter((t) => t.id !== action.id);
    }
    default: {
      throw Error("Unknown action: " + action.type);
    }
  }
}
```

일반적으로는 switch 문을쓰며, case명으로 이벤트 핸들러에서 일어날 일을 지정합니다.

### 컴포넌트에서 reducer 사용하기.

```jsx
import { useReducer } from 'react';

const [tasks, dispatch] = useReducer(reducer 함수
, 초기 state 값);
```

기본 형태는 위와 같고, 이를 통해 state를 담을 수 있는 값, dispatch함수를 반환합니다.

```jsx
import { useReducer } from "react";
import AddTask from "./AddTask.js";
import TaskList from "./TaskList.js";

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

  function handleAddTask(text) {
    dispatch({
      type: "added",
      id: nextId++,
      text: text,
    });
  }

  function handleChangeTask(task) {
    dispatch({
      type: "changed",
      task: task,
    });
  }

  function handleDeleteTask(taskId) {
    dispatch({
      type: "deleted",
      id: taskId,
    });
  }

  return (
    <>
      <h1>Prague itinerary</h1>
      <AddTask onAddTask={handleAddTask} />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}

function tasksReducer(tasks, action) {
  switch (action.type) {
    case "added": {
      return [
        ...tasks,
        {
          id: action.id,
          text: action.text,
          done: false,
        },
      ];
    }
    case "changed": {
      return tasks.map((t) => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case "deleted": {
      return tasks.filter((t) => t.id !== action.id);
    }
    default: {
      throw Error("Unknown action: " + action.type);
    }
  }
}

let nextId = 3;
const initialTasks = [
  { id: 0, text: "Visit Kafka Museum", done: true },
  { id: 1, text: "Watch a puppet show", done: false },
  { id: 2, text: "Lennon Wall pic", done: false },
];
```

위의 코드에선 tasksReducer를 통하여 state를 관리하게됩니다.

## useState와 useReducer 비교하기

1. 코드크기
   => useState는 미리 작성해야 하는 코드를 줄일 수 있습니다.
   => useReducer는 기본 작성량은 많으나, 이벤트 핸들러가 여러개일 경우, 코드의 양을 줄여줍니다.

2.가독성
=>useState의 경우, 간단한 state 업데이트의 경우 가독성이 좋습니다
=> useReducer는 단순한 구조보단, 복잡한 구조의 컴포넌트에서 이벤트핸들러의 역할을 명확히 할 수 있어 효율적입니다.

3.디버깅
=>useState의 경우 버그의 소재지를 찾기 어렵습니다
=>useReducer의 경우, 각 단계에 console을 통해 버그의 소재지 파악이 명확합니다.
