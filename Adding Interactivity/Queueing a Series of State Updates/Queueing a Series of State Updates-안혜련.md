# Queueing a Series of State Updates
"state 업데이트 큐와 batching"

>**State 업데이트 = 새로운 렌더링 요청**

- `setState`를 호출해도 기존 렌더링의 변수 값은 바뀌지 않고 새로운 렌더링을 요청한다.
- `setState`는 변경된 `state` 값을 다음 렌더링에서만 적용한다.

```jsx
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 1);
        setNumber(number + 1);
        setNumber(number + 1);
      }}>+3</button>
    </>
  )
}

```

각 렌더링의 state 값은 고정되어 있으므로, 

첫 번째 렌더링의 이벤트 핸들러의 `number` 값은 `setNumber(1)`을 몇 번 호출하든 항상 0..!!!



+. **React는 state 업데이트를 하기 전에 이벤트 핸들러의 모든 코드가 실행될 때까지 기다린다.**

 이 때문에 리렌더링은 모든 `setNumber()` 호출이 완료된 이후에만 일어난다.

- **Batching이란?**
    - React는 이벤트 핸들러의 모든 코드가 실행된 후, `state` 업데이트를 한 번에 처리
    - 여러 `setState` 호출이 있어도 React는 **한 번의 리렌더링**만 발생시켜 성능을 최적화

## 동일한 `state`를 여러 번 업데이트하는 방법
한 번의 렌더링에서 state를 누적해서 업데이트하려면 어떻게 할까

=>  **기존 값으로 업데이트 (`업데이터 함수` 사용)**
   
  - `setState`에 함수 형태 (`updater function`)를 전달하면 이전 `state`를 기반으로 계산한다.
    
    ```jsx
    
    setNumber(n => n + 1); // n은 이전 state 값을 의미
    ```
    
이전 큐의 state를 기반으로 다음 state를 계산하는 함수를 전달할 수 있다

## **업데이트 순서 처리**

- React는 `state` 업데이트 요청을 큐에 쌓고, 한 번에 처리한다.

```jsx

setNumber(0 + 5); // "5로 변경"
setNumber(n => n + 1); // 이전 값에 1 더하기
setNumber(42); // "42로 변경"

```

- React는 큐를 순회하며 최종 `state`를 계산한다.:
1.  "5로 변경" → `state = 5`
2. `n => n + 1` → `state = 6`
3. "42로 변경" → `state = 42`

결론적으로, 마지막 업데이트가 값을 덮어쓰므로 최종 값은 항상 42가 된다.

---

### ** 이벤트 핸들러에서 최신 `state` 계산**

- **업데이터 함수 사용 예시**:
    
    ```jsx
    
    <button onClick={() => {
      // setNumber(number + 1); 기존방식 
      setNumber(n => n + 1);
      setNumber(n => n + 1);
      setNumber(n => n + 1);
    }}>+3</button>
    
    ```
    
    - 결과: `number`는 3 증가.
    - React는 이벤트 핸들러 종료 후, 큐를 순회하여 최종 값을 계산

---
#### state 업데이트는 단순한 값 변경이 아니라 **큐를 통해 배치(batch) 처리된다는 점** 이 중요!!

1. 이전 `state` 값을 기반으로 업데이트하려면 setNumber(n => n + 1) `업데이터 함수`를 사용해야 한다.

2. 최신 상태를 정확히 반영하려면 `state`를 대체하는 대신 업데이트하는 방식을 사용하자!!!