## Reducer와 Context로 상태 관리

### 1. Reducer와 context 결합하기

여러 컴포넌트가 상태를 반영,변경하려면 state,state를 변경하는 이벤트 핸들러를 명시적으로 props로 전달해야 함
대신 state, dispatch함수를 context에 넣어 트리에서 모든 컴포넌트들이 props drilling 없이 state와 관련 로직을 사용할 수 있게 할 수 있음

```jsx
// TaskContext.js
// ✅ 먼저 state, dispatch context 생성
import { createContext, useReducer } from 'react';

const TasksContext = createContext(null);
const TasksDispatchContext = createContext(null);

export function useTasks() {
  return useContext(TasksContext);
}
export function useTasksDispatch() {
  return useContext(TasksDispatchContext);
}

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

// App.js
// ✅ 최상위 컴포넌트에서 두 context를 가져와
// useReducer에서 반환된 tasks, dispatch를 트리 전체에 제공함
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';
import { TasksProvider } from './TasksContext.js';

export default function TaskApp() {
  return (
    <TasksProvider>
        <h1>Todos</h1>
        <AddTask/> {/* 이벤트 핸들러, state를 props로 넘기지 않아도 됨 */}
        <TaskList/>
    </TasksProvider>
  );
}

// TaskList.js
import { useState, useContext } from 'react';
import { useTasks, useTasksDispatch } from './TasksContext.js';

export default function TaskList() {
  // ✅ 하위 컴포넌트에선 필요한 context를 사용함
  const tasks = useTasks();
  ...
}

// AddTask.js
import { useState, useContext } from 'react';
import { useTasksDispatch } from './TasksContext.js';

export default function AddTask() {
  const [text, setText] = useState('');
  const dispatch = useTasksDispatch(); 
  ...
  return (
    ...
    <button onClick={() => {
      setText('');
      dispatch({
        type: 'added',
        id: nextId++,
        text: text,
      });
    }}>Add</button>
    ...
  )
}
```