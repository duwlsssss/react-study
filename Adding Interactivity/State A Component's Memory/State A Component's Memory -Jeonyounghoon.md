# State에 대하여

어떤 데이터가 바뀌면서 UI가 변경됩니다. 그 데이터를 기억할 수 있는 메모리를 state라고 부릅니다.

## 컴포넌트 업데이트

컴포넌트를 새로 업데이트 하려면 1. 렌더링 사이에 데이터를 유지하고, 2. 리액트가 새로운 데이터로 컴포넌트를 재 렌더링 하도록 유발해야합니다. => 이걸 useState가 해줍니다.

## state의 작동 방식

const [index, setIndex] = useState(0);

1. 컴포넌트가 처음 초기값으로 렌더링 됩니다. (리액트가 index값 기억)
2. state가 setter함수에 의해 업데이트됩니다. (업데이트된 값 기억)
3. 컴포넌트가 두 번째로 렌더링 됩니다.
   반복

## state의 특징

동일한 컴포넌트를 두 번 렌더링한다면 각 복사본은 완전히 독립적인 state를 가집니다  
import React, { useState } from 'react';

```jsx
function Counter({ label }) {
  const [count, setCount] = useState(0);

  return (
    <div>
      <h3>{label}</h3>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increase</button>
    </div>
  );
}

export default function App() {
  return (
    <div>
      <h1>Independent State Example</h1>
      <Counter label="Counter 1" />
      <Counter label="Counter 2" />
    </div>
  );
}
```
