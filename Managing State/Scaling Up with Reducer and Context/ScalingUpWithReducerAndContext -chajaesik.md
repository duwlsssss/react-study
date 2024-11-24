#### Reducer를 사용하면 컴포넌트의 state 업데이트 로직을 통합할 수 있다.

#### Context를 사용하면 다른 컴포넌트들에 정보를 전달 할 수 있다.

#### 따라서 Reducer와 Context를 함께 사용하면 복잡한 화면의 state를 "쉽계" 관리 할 수 있다.

 

### 학습 내용
- reducer와 context를 결합하는 방법
- state와 dispatch 함수를 prop으로 전달하지 않는 방법
- context와 state 로직을 별도의 파일에서 관리한느 방법
 

#### 이전 포스트에서 Reducer와 Context를 사용하는 경우와 그 이유에 대해 알아보았으니 이번에는 생략하고

- Reducer: https://vitamin3000.tistory.com/128

- Context : https://vitamin3000.tistory.com/129
#### 바로 방법으로 넘어가겠다.


### Reducer와 Context를 결합하기
- Context를 생성
- State와 dispatch 함수를 context에 넣는다
- 트리 안에서 context를 사용

### 코드 예시와 함께 자세히 살펴보자
 

### 1. Context 생성

 

#### useReducer Hook은 tasks와 업데이트 할 수 있는 dispatch 함수를 반환한다.

 
```javascript
const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
 ```

#### 트리로 전달하려면, 두 개의 별개의 context를 생성해야 한다.

 

- TasksContext는 현재 tasks 리스트를 제공한다.
- TaskDispatchContext는 컴포넌트에서 action을 dispatch하는 함수를 제공

#### 왜 tasks 리스트와 action을 제공하는 context를 생성해야 하는가?
#### -> useReducer에서 사용하는 매개변수의 첫번째가 현재의 state값과 , 두번째가 action 객체이기 때문이다.

```javascript
import { createContext } from 'react';

export const TasksContext = createContext(null);
export const TasksDispatchContext = createContext(null);
 ```

 

### 2. State와 dispatch 함수를 context에 넣기
```javascript
import { TasksContext, TasksDispatchContext } from './TasksContext.js';

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
  // ...
  return (
    <TasksContext.Provider value={tasks}>
      <TasksDispatchContext.Provider value={dispatch}>
        ...
      </TasksDispatchContext.Provider>
    </TasksContext.Provider>
  );
}
```
#### 이때 TasksContext.Provider등 Provider로 "감싸"줘야 React가 Context의 값을 추적할 수 있다.

 

 

### 3. 트리 안에서 Context 사용하기
 
```javascript
return (
    <TasksContext.Provider value={tasks}>
      <TasksDispatchContext.Provider value={dispatch}>
        <h1>Day off in Kyoto</h1>
        <AddTask
          onAddTask={handleAddTask} // prop으로 전달
        />
        <TaskList
          tasks={tasks}
          onChangeTask={handleChangeTask} // prop으로 전달
          onDeleteTask={handleDeleteTask} // prop으로 전달
        />
      </TasksDispatchContext.Provider>
    </TasksContext.Provider>
  );
 ```

#### 이제 Context를 사용하기에, tasks 리스트나 이벤트 핸들러를 트리 아래로 "내려" 전달할 필요가 없다.

 
```javascript
<TasksContext.Provider value={tasks}>
  <TasksDispatchContext.Provider value={dispatch}>
    <h1>Day off in Kyoto</h1>
    <AddTask />
    <TaskList />
  </TasksDispatchContext.Provider>
</TasksContext.Provider>
 ```

#### 만약 tasks 리스트를 업데이트하기 위해서 컴포넌트에서 context의 dispatch 함수를 읽고 호출할 수 있다.

 
```javascript
export default function AddTask({ onAddTask }) {
  const [text, setText] = useState('');
  const dispatch = useContext(TasksDispatchContext);
  // ...
  return (
    // ...
    <button onClick={() => {
      setText('');
      dispatch({
        type: 'added',
        id: nextId++,
        text: text,
      });
    }}>Add</button>
    // ...
``` 

#### State는 여전히 최상위 TaskApp 컴포넌트에서 useReducer로 관리되고 있다.

#### 그러나 이제 context를 가져와 트리 아래의 모든 컴포넌트에서 해당 tasks 및 dispatch를 사용할 수 있다.

 

### 하나의 파일로 합치기
#### 반드시 그럴 필요는 없지만... reducer와 context를 모두 하나의 파일에 작성하면 컴포넌트들을 조금 더 정리할 수 있다.

 

#### 현재 Context파일에서는 다음과 같이 두 개의 context만을 선언하고 있다.

 

```javascript
import { createContext } from 'react';

export const TasksContext = createContext(null);
export const TasksDispatchContext = createContext(null);
 ```

### Reducer와 Context를 합치려면...

 

#### Reducer를 같은 파일로 옮기고 모든 것을 하나로 묶는 TasksProvider 컴포넌트를 새로 선언한다.

 
```javascript
export function TasksProvider({ children }) { // children을 prop으로 받는다.
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

  return (
    <TasksContext.Provider value={tasks}>
      <TasksDispatchContext.Provider value={dispatch}>
        {children} // JSX 전달
      </TasksDispatchContext.Provider>
    </TasksContext.Provider>
  );
}
```

### 위 내용을 정리하자면 ..

- Reducer로 state를 관리
- 두 context를 모두 하위 컴포넌트에 제공
- children을 prop으로 받기 때문에 JSX를 전달할 수 있다
 

 

### Context 관리 파일에서 use 함수들도 내보낼 수 있다.

```javascript
export function useTasks() {
  return useContext(TasksContext);
}

export function useTasksDispatch() {
  return useContext(TasksDispatchContext);
}

// 다른 파일에서 사용하려면..
import { tasks, dispatch } from 경로
const tasks = useTasks();
const dispatch = useTasksDispatch();
 ```

#### Context-Reducer 조합을 앱에 여러 개 만들 수 있다.