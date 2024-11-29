# Reducer와 Context로 앱 확장하기
Reducer를 사용하면 컴포넌트의 state 업데이트 로직을 통합할 수 있습니다. Context를 사용하면 다른 컴포넌트들에 정보를 전달할 수 있습니다. Reducer와 context를 함께 사용하여 복잡한 화면의 state를 관리할 수 있습니다.

## 1. Reducer와 Context 결합의 필요성
### Reducer의 역할
- 상태 업데이트 로직을 한곳에 모아 관리.
- 이벤트 핸들러를 간결하게 유지.
### Context의 역할
- 데이터를 컴포넌트 트리의 모든 자손에게 전달.
- prop drilling을 제거하여 코드 단순화.
### 조합의 이점
- Reducer는 상태 관리와 로직을 담당.
- Context는 상태(state)와 dispatch를 하위 컴포넌트에 쉽게 전달.

### Step 1: Context 생성
1. TasksContext → 상태 값을 전달.
2. TasksDispatchContext → dispatch 함수를 전달.
``` javascript

import { createContext } from 'react';

export const TasksContext = createContext(null);
export const TasksDispatchContext = createContext(null);
```
### Step 2: Context에 State와 Dispatch 제공
useReducer를 사용하여 상태와 dispatch 함수를 생성한 뒤, 두 Context를 통해 이를 트리 전체에 제공.

``` javascript

import { useReducer } from 'react';
import { TasksContext, TasksDispatchContext } from './TasksContext';

function TaskApp() {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

  return (
    <TasksContext.Provider value={tasks}>
      <TasksDispatchContext.Provider value={dispatch}>
        {/* 하위 컴포넌트 */}
      </TasksDispatchContext.Provider>
    </TasksContext.Provider>
  );
}
```

### Step 3: 하위 컴포넌트에서 Context 사용
하위 컴포넌트에서는 useContext를 사용하여 필요한 데이터(tasks, dispatch)를 가져옴.

예: TaskList 컴포넌트

``` javascript

import { useContext } from 'react';
import { TasksContext } from './TasksContext';

function TaskList() {
  const tasks = useContext(TasksContext);

  return (
    <ul>
      {tasks.map(task => (
        <li key={task.id}>{task.text}</li>
      ))}
    </ul>
  );
}
```
``` javascript
import { useContext, useState } from 'react';
import { TasksDispatchContext } from './TasksContext';

function AddTask() {
  const [text, setText] = useState('');
  const dispatch = useContext(TasksDispatchContext);

  const handleAddTask = () => {
    dispatch({ type: 'added', text });
    setText('');
  };

  return (
    <div>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <button onClick={handleAddTask}>Add</button>
    </div>
  );
}
```

## 3. 최적화: Context와 Reducer 통합

TasksContext.js에 상태, reducer, 그리고 provider를 모두 통합하여 관리.

TasksProvider 컴포넌트
- useReducer로 상태 관리.
- Context Provider를 통해 tasks와 dispatch를 제공.
``` javascript

import { createContext, useReducer, useContext } from 'react';

// Context 생성
export const TasksContext = createContext(null);
export const TasksDispatchContext = createContext(null);

// Reducer
function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added':
      return [...tasks, { id: Date.now(), text: action.text, done: false }];
    case 'changed':
      return tasks.map(task =>
        task.id === action.task.id ? action.task : task
      );
    case 'deleted':
      return tasks.filter(task => task.id !== action.id);
    default:
      throw new Error(`Unknown action: ${action.type}`);
  }
}

// Provider 컴포넌트
export function TasksProvider({ children }) {
  const [tasks, dispatch] = useReducer(tasksReducer, []);

  return (
    <TasksContext.Provider value={tasks}>
      <TasksDispatchContext.Provider value={dispatch}>
        {children}
      </TasksDispatchContext.Provider>
    </TasksContext.Provider>
  );
}

// Custom Hooks
export function useTasks() {
  return useContext(TasksContext);
}

export function useTasksDispatch() {
  return useContext(TasksDispatchContext);
}
```

사용법
이제 TasksProvider를 사용하여 컴포넌트 트리를 감싸고, useTasks와 useTasksDispatch를 통해 데이터를 읽고 수정.

``` javascript

import { TasksProvider, useTasks, useTasksDispatch } from './TasksContext';

function TaskApp() {
  return (
    <TasksProvider>
      <h1>Task Management</h1>
      <AddTask />
      <TaskList />
    </TasksProvider>
  );
}

function TaskList() {
  const tasks = useTasks();

  return (
    <ul>
      {tasks.map(task => (
        <li key={task.id}>{task.text}</li>
      ))}
    </ul>
  );
}

function AddTask() {
  const dispatch = useTasksDispatch();
  const [text, setText] = useState('');

  const handleAddTask = () => {
    dispatch({ type: 'added', text });
    setText('');
  };

  return (
    <div>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <button onClick={handleAddTask}>Add</button>
    </div>
  );
}
```

## 4. 사용자 정의 Hook의 장점
useTasks와 useTasksDispatch로 Context를 간편하게 사용할 수 있음

- Context 내부 구조를 몰라도 상태를 읽고 수정 가능.
- 코드 재사용성과 가독성 증가.
