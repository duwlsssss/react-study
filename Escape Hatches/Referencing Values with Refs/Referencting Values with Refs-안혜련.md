

# useRef란 무엇인가?
- useRef는 React에서 제공하는 Hook으로, 컴포넌트의 렌더링과 무관한 값을 기억하는 데 사용됩니다.
- 주로 렌더링을 유발하지 않으면서 값을 저장하거나, DOM 요소에 접근할 때 사용됩니다.

``` jsx
import { useRef } from 'react';

const ref = useRef(0); // 초기값을 0으로 설정
console.log(ref.current); // { current: 0 }
```

- useRef는 { current: initialValue } 형태의 객체를 반환합니다.
- ref.current를 통해 값을 읽고 수정할 수 있습니다.
## useRef와 useState의 차이
### useRef 
- ref.current를 직접 수정 가능
- ref.current를 변경해도 렌더링이 발생하지 않음
- 렌더링과 무관한 값을 저장하거나 DOM요소에 접근할 떄 사용
- useRef(initialValue로 설정)
- 상태가 항상 최신값으로 유지

### useState
- 반드시 setState를 통해 변경
- setState를 호출하면 렌더링이 발생
- 렌더링에 영향을 미치는 값 관리
- useState(initialValue)로 설정
- 렌더링 마다 고유한 상태 스냅샷을 가짐

## 언제 useRef를 사용해야 하는가?
1. 렌더링을 유발하지 않고 값 유지:
- setInterval, timeout ID, 이전 값 비교 등과 같은 경우.
2. DOM 요소 접근:
- 특정 DOM 요소에 직접 접근하여 조작할 때.

###  useRef로 상태 유지
버튼 클릭 횟수 저장
``` javascript

import { useRef } from 'react';

export default function Counter() {
  const countRef = useRef(0);

  function handleClick() {
    countRef.current += 1;
    alert(`Clicked ${countRef.current} times`);
  }

  return <button onClick={handleClick}>Click me!</button>;
}
```
- countRef.current 값은 증가하지만 컴포넌트는 다시 렌더링되지 않습니다.
### useRef와 state의 차이점 확인
``` javascript

import { useState } from 'react';

export default function CounterWithState() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1); // 렌더링 발생
  }

  return <button onClick={handleClick}>Clicked {count} times</button>;
}
```
- etState를 호출하면 컴포넌트가 다시 렌더링됩니다.
### useRef로 setInterval ID 저장
#### 스톱워치 예제
- useRef로 setInterval의 ID를 저장해 관리.
``` javascript
코드 복사
import { useState, useRef } from 'react';

export default function Stopwatch() {
  const [startTime, setStartTime] = useState(null);
  const [now, setNow] = useState(null);
  const intervalRef = useRef(null); // interval ID 저장

  function handleStart() {
    setStartTime(Date.now());
    setNow(Date.now());

    clearInterval(intervalRef.current); // 기존 interval 정리
    intervalRef.current = setInterval(() => {
      setNow(Date.now());
    }, 10);
  }

  function handleStop() {
    clearInterval(intervalRef.current); // interval 정리
  }

  const secondsPassed = startTime && now ? (now - startTime) / 1000 : 0;

  return (
    <>
      <h1>Time passed: {secondsPassed.toFixed(3)}</h1>
      <button onClick={handleStart}>Start</button>
      <button onClick={handleStop}>Stop</button>
    </>
  );
}
```
## useRef 사용 시 주의사항
1. 렌더링 중 ref.current를 읽거나 쓰지 마세요:
    - React의 예측 가능한 렌더링 흐름을 깨트릴 수 있습니다.
    - 렌더링에 사용되는 값은 항상 state로 관리하세요.
2. 렌더링과 무관한 값만 ref에 저장:
    - DOM 엘리먼트 접근, 타이머 ID, 이전 값 기록 등.
## useRef를 사용하는 이유
- 렌더링 효율성: useRef는 값을 변경해도 컴포넌트가 다시 렌더링되지 않습니다.
- 외부 시스템과의 통합: 브라우저 API(DOM 접근 등)와 통합 작업 시 유용합니다.

useRef는 React의 **단방향 데이터 흐름에서 벗어나는 "탈출구"**로 사용되며, 필요한 경우에만 적절히 사용해야 합니다. Refs를 남용하면 코드의 유지보수성이 떨어질 수 있다 !




