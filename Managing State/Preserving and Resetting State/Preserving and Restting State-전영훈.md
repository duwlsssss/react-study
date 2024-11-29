# State를 보존하고 초기화하기

## React는 UI 트리에서의 '위치'를 통해 각 state가 어떤 컴포넌트에 속하는지 추적합니다

=>따라서 리렌더링(state의 업데이트)마다 언제 state를 보존하고 또 state를 초기화할지 컨트롤할 수 있습니다.

```jsx
import { useState } from "react";

export default function App() {
  return (
    <div>
      <Counter />
      <Counter />
    </div>
  );
}

function Counter() {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = "counter";
  if (hover) {
    className += " hover";
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>Add one</button>
    </div>
  );
}
```

특정 카운터가 생긴되면 해당 컴포넌트의 상태만 갱신됩니다
다시, 여기서 우리는 리액트가 트리에서 그 위치의 state를 추적하여 그 state만 갱신함을 볼 수 있습니다.
=> 리액트는 어느 한 컴포넌트가 사라짐에 따라 트리에서 그 컴포넌트가 사라지면, 자연스럽게 그 컴포넌트의 state도 같이 없앱니다.

반대로, 그 컴포넌트의 조건부 상태가 어떻게 되는지 상관없이 컴포넌트가 트리 내에서 같은 자리에 위치한다면 state를 유지합니다.

다만, 여기서 우리는 그 위치의 개념이 JSX 마크업에서가 아닌 UI 트리에서의 위치라는 것을 명심해야합니다 ( React는 함수 안 어디에 조건문이 있는지 모릅니다. React는 당신이 반환하는 트리만 “봅니다”)

```jsx
export default function App() {
  const [isFancy, setIsFancy] = useState(false);
  if (isFancy) {
    return (
      <div>
        <Counter isFancy={true} />
        <label>
          <input
            type="checkbox"
            checked={isFancy}
            onChange={e => {
              setIsFancy(e.target.checked)
            }}
          />
          Use fancy styling
        </label>
      </div>
    );
  }
```

가령 여기서의 조건부로 나뉜 Counter 컴포넌트는 트리상 동일한 한 위치에 존재합니다. => 이걸 판단하는 중요한 기준은

1. 같은 부모 아래에 있고, 같은 순서에 렌더링되는 경우.
2. 컴포넌트의 **키 (key)**가 동일한 경우.

가 있는데, 해당 코드의 경우 <Counter>는 동일한 부모 <div>의 첫 번째 자식으로 렌더링됩니다.

## 같은 위치의 다른 컴포넌트는 state를 초기화합니다

```jsx
{
  isPaused ? <p>See you later!</p> : <Counter />;
}
```

해당 코드의 경우, IsPaused의 값에 따라 <p>요소와 <Counter>컴포넌트가 서로 교체되는데, 교체되는 것은 트리의 내용의 변화로 간주하여 Counter 내에 존재했던 state가 사라지게 됩니다.

요소와 컴포넌트 뿐만 아니라, 컴포넌트를 감싸고 있던 요소가 달라져도, 그 말은 곧 트리의 구조가 달라진다는 말이기에 마찬가지로 state가 초기화됩니다.

즉, 리렌더링할 때 state를 유지하고 싶다면, 트리 구조가 “같아야” 합니다

## 같은 위치에서 state를 초기화하기

위의 내용을 토대로 다른 위치에 컴포넌트를 렌더링 하면 됩니다.

````jsx
 {isPlayerA ? (
        <Counter person="Taylor" />
      ) : (
        <Counter person="Sarah" />
      )}
      ```
````

이 경우, isPlayerA의 값에 따라 한 컴포넌트만 렌더링됩니다.
=> 삼항 연산자는 단 하나의 반환값만을 생성하기 때문에 <Counter> 컴포넌트는 항상 동일한 부모의 첫 번째 자식으로 렌더링됩니다

```jsx
{
  isPlayerA && <Counter person="Taylor" />;
}
{
  !isPlayerA && <Counter person="Sarah" />;
}
```

이 경우, isPlayerA의 값에 따라 각각 다른 조건에서 렌더링됩니다. => 별도의 노드로 간주합니다

또 다른 조건으론 key인데, react는 key를 바꾸면, DOM 요소를 없애는 특징이 있어서 key값을 변형시키면 해당 state가 자연스럽게 컴포넌트가 삭제되면서 같이 삭제됩니다.
