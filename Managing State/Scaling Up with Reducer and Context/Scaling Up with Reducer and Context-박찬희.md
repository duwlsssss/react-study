# Reducer와 Context로 앱 확장하기
복잡한 state를 관리해야하는 경우 Reducer를 통해 복잡한 state와 state를 다루는 이벤트 핸들러들을 통합해서 관리하고 context를 통해 props drilling이 발생하지 않도록 하위 컴포넌트들에게 props로 state와 이벤트 핸들러를 전달할 수 있습니다.

## Reducer와 context를 결합하기
Reducer와 Context를 결합하는 방법을 3단계로 나눠서 알아봅시다.

### 1. Context생성
`useReducer` Hook은 현재 `tasks`와 업데이트할 수 있는 `dispatch` 함수를 반환합니다.
```jsx
//App.js
const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
```
트리를 통해 전달하려면, 두 개의 별개의 context를 생성해야합니다.
- `TaskContext`는 현재 `tasks`리스트를 제공
- `TaskDispatchContext`는 컴포넌트에서 action을 dispatch하는 함수를 제공합니다.
```jsx
// TaskContext.ts
import { createContext } from 'react';

export const TasksContext = createContext(null);
export const TasksDispatchContext = createContext(null);
```
여기서 두 context에 모두 기본값으로 null을 전달하고 있습니다. 실제 값은 `TaskApp`컴포넌트에 제공될 것입니다.

### 2. State와 dispatch 함수를 context에 넣기

`TaskApp` 컴포넌트에서 두 context를 가져올 수 있습니다. `useReducer()`에서 반환 된 `tasks` 및 `dispatch`를 가져와 아래 트리 전체에 제공하세요.

```jsx
// App.js

import { useReducer } from 'react';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';
import { TasksContext, TasksDispatchContext } from './TasksContext.js';

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
  // ...
  return (
    <TasksContext.Provider value={tasks}>
      <TasksDispatchContext.Provider value={dispatch}>
        <h1>Day off in Kyoto</h1>
        <AddTask
          onAddTask={handleAddTask}
        />
        <TaskList
          tasks={tasks}
          onChangeTask={handleChangeTask}
          onDeleteTask={handleDeleteTask}
        />
      </TasksDispatchContext.Provider>
    </TasksContext.Provider>
  );
}
```
지금은 props와 context 모두를 이용하여 정보를 전달하고 있습니다.

### 3. 트리 안에서 context 사용하기
이제 prop을 통한 전달을 제거합니다. `tasks` 리스트나 이벤트 핸들러를 트리 아래로 전달할 필요가 없습니다.<br/>
대신 필요한 컴포넌트에서는 `TaskContext`에서 `tasks`리스트를 읽을 수 있습니다.
또한 `tasks` 리스트를 업데이트 하기 위해서 컴포넌트에 context의 `dispatch`함수를 읽고 호출할 수 있습니다.
```jsx
//App.js
import { useReducer } from 'react';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';
import { TasksContext, TasksDispatchContext } from './TasksContext.js';

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
  // ...
  return (
    <TasksContext.Provider value={tasks}>
      <TasksDispatchContext.Provider value={dispatch}>
        <h1>Day off in Kyoto</h1>
        <AddTask/> // props 제거 
        <TaskList/> // props 제거
      </TasksDispatchContext.Provider>
    </TasksContext.Provider>
  );
}
// `TaskApp` 컴포넌트는 어떤 이벤트 핸들러도 아래로 전달하지 않음
//AddTask.js
import { useState, useContext } from 'react';
import { TasksDispatchContext } from './TasksContext.js';

export default function AddTask({ onAddTask }) {
  const [text, setText] = useState('');

  // `tasks` 리스트 업데이트를 위한 dispatch함수
  const dispatch = useContext(TasksDispatchContext);
  return (
    <>
      <input
        placeholder="Add task"
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <button onClick={() => {
        setText('');
        //dispatch 함수 호출!
        dispatch({
          type: 'added',
          id: nextId++,
          text: text,
        }); 
      }}>Add</button>
    </>
  );
}

let nextId = 3;

// TaskList.js
import { useState, useContext } from 'react';
import { TasksContext, TasksDispatchContext } from './TasksContext.js';

export default function TaskList() {
  const tasks = useContext(TasksContext);
  return (
    <ul>
      {tasks.map(task => (
        <li key={task.id}>
          <Task task={task} />
        </li>
      ))}
    </ul>
  );
}

function Task({ task }) {
  const [isEditing, setIsEditing] = useState(false);
  const dispatch = useContext(TasksDispatchContext); 
  let taskContent;
  if (isEditing) {
    taskContent = (
      <>
        <input
          value={task.text}
          onChange={e => {
            dispatch({
              type: 'changed',
              task: {
                ...task,
                text: e.target.value
              }
            });
          }} />
        <button onClick={() => setIsEditing(false)}>
          Save
        </button>
      </>
    );
  } else {
    taskContent = (
      <>
        {task.text}
        <button onClick={() => setIsEditing(true)}>
          Edit
        </button>
      </>
    );
  }
  return (
    <label>
      <input
        type="checkbox"
        checked={task.done}
        onChange={e => {
          dispatch({
            type: 'changed',
            task: {
              ...task,
              done: e.target.checked
            }
          });
        }}
      />
      {taskContent}
      <button onClick={() => {
        dispatch({
          type: 'deleted',
          id: task.id
        });
      }}>
        Delete
      </button>
    </label>
  );
}
//`TaskList`도 `Task` 컴포넌트로 이벤트 핸들러를 전달하지 않습니다. 각 컴포넌트는 필요한 context를 읽습니다.
```

State는 여전히 최상위 `TaskApp`컴포넌트에서 `useReducer`로 관리되고 있습니다. 또한 이제 context를 가져와 트리 아래의 모든 컴포넌트에 해당 `tasks` 및 `dispatch`를 사용할 수 있습니다.

## 하나의 파일로 합치기
반드시 이런 방식으로 작성하지 않아도 되지만, reducer와 context를 모두 하나의 파일에 작성하면 컴포넌트들을 조금 더 정리할 수 있습니다. 현재, `TasksContext.js`는 두 개의 context만을 선언하고 있습니다.
```jsx
import { createContext } from 'react';

export const TasksContext = createContext(null);
export const TasksDispatchContext = createContext(null);
```
Reducer를 같은 파일로 옮기고 `TasksProvider`컴포넌트를 새로 선언합니다. 이 컴포넌트는 모든 것을 하나로 묶는 역할을 하게 됩니다.
1. Reducer로 state를 관리합니다.
2. 두 context를 모두 하위 컴포넌트에 제공합니다.
3. children을 prop으로 받기 때문에 JSX를 전달할 수 있습니다.

```jsx
//TasksContext.js
import { createContext, useReducer } from 'react';

export const TasksContext = createContext(null);
export const TasksDispatchContext = createContext(null);

export function TasksProvider({ children }) {
  const [tasks, dispatch] = useReducer(
    tasksReducer,
    initialTasks
  );

  return (
    <TasksContext.Provider value={tasks}>
      <TasksDispatchContext.Provider value={dispatch}>
        {children}
      </TasksDispatchContext.Provider>
    </TasksContext.Provider>
  );
}
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

const initialTasks = [
  { id: 0, text: 'Philosopher’s Path', done: true },
  { id: 1, text: 'Visit the temple', done: false },
  { id: 2, text: 'Drink matcha', done: false }
];

```
이렇게 하면 `TaskApp` 컴포넌트의 복잡성과 연결이 모두 제거됩니다.
`TasksContext.js`에서 context를 사용하기 위한 use함수들도 내보낼 수 있습니다.
```jsx
export function useTasks() {
  return useContext(TasksContext);
}

export function useTasksDispatch() {
  return useContext(TasksDispatchContext);
}

// 이 함수를 사용하여 컴포넌트에서 context를 읽을 수 있습니다.
const tasks = useTasks();
const dispatch = useTasksDispatch();
```
위 예시는 동작이 바뀌는 것은 아니지만 다음에 context를 더 분리하거나 함수들에 로직을 추가하기 쉬워집니다.

`useTasks`와 `useTasksDispatch` 같은 함수들을 사용자 정의 Hook이라고 합니다. 사용자 정의 Hook에서도 `useContext`같은 다른 Hook을 사용 가능합니다.