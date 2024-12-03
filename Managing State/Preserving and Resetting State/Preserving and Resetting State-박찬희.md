# state를 보존하고 초기화하기

각 컴포넌트는 독립된 state를 가집니다. 
React는 UI트리에서의 위치를 통해 각 state가 어떤 컴포넌트에 속하는지 추적합니다.
리랜더링마다 언제 state를 보존하고 초기화할지 컴트롤 할 수 있습니다.

## state는 랜더트리의 위치에 연결됩니다.

React는 UI안에 있는 컴포넌트 구조로 랜더 트리를 만듭니다.
컴포넌트에 state를 줄 때 state가 컴포넌트 안에 살고있다고 생각 할 수도 있습니다. 하지만 사실 state는 React 안에 있습니다.
React는 컴포넌트가 UI 트리에 있는 위치를 이용해 React가 가지고 있는 각state를 알맞은 컴포넌트에 연결합니다.
```jsx
import { useState } from 'react';

export default function App() {
  const counter = <Counter />;
  return (
    <div>
      {counter}
      {counter}
    </div>
  );
}
function Counter() {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Add one
      </button>
    </div>
  );
}
```
동일한 `<Counter/>` JSX 태그가 다른 두 군데에서 랜더링 되고 있습니다. 이 둘은 각각 트리에서 자기 고유의 위치에 랜더링되어 있으므로 분리되어 있는 카운터 입니다. 일반적으로 React를 사용할 때 위치에 대해 생각할 필요는 없지만 React가 어떻게작동하는지 이해할 때 유용합니다.

React에서 화면의 각 컴포넌트는 완전히 분리된 state를 가집니다. 
React는 트리의 동일한 컴포넌트를 동일한 위치에서 랜더링 하는 동안 상태를 유지합니다. 컴포넌트가 state를 가지고있더라도 외부에서 랜더링 된 컴포넌트를 제거 한 후 다시 생성한다면 state는 초기화됩니다.
즉 **React는 컴포넌트가 UI트리에서 그 자리에 랜더링되는 한 state를 유지합니다.**

## 같은자리의 같은 컴포넌트는 state를 보존합니다.
```jsx
import { useState } from 'react';

export default function App() {
  const [isFancy, setIsFancy] = useState(false);
  return (
    <div>
      {isFancy ? (
        <Counter isFancy={true} /> 
      ) : (
        <Counter isFancy={false} /> 
      )}
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

function Counter({ isFancy }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }
  if (isFancy) {
    className += ' fancy';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Add one
      </button>
    </div>
  );
}
```
이 예제는 서로 다른 두 `<Counter/>`태그가 있습니다. 하지만 체크 박스를 선택하거나 선택 해제할 때 카운터 state는 초기화되지 않습니다. `isFancy`가 `true`이거나 `false`여도 `<Counter/>`는 root `App`컴포넌트가 반환한 `div`의 첫번째 자식으로 있기 때문입니다.

## 같은 위치의 다른 컴포넌트는 state를 초기화합니다.

```jsx
import { useState } from 'react';

export default function App() {
  const [isPaused, setIsPaused] = useState(false);
  return (
    <div>
      {isPaused ? (
        <p>See you later!</p> 
      ) : (
        <Counter /> 
      )}
      <label>
        <input
          type="checkbox"
          checked={isPaused}
          onChange={e => {
            setIsPaused(e.target.checked)
          }}
        />
        Take a break
      </label>
    </div>
  );
}

function Counter() {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Add one
      </button>
    </div>
  );
}
```
이번 예제에서는 다른 컴포넌트 타입으로 바꿉니다. `<div>`가 `Counter`에서 `p`로 자식이 바뀌면서 React는 UI트리에서 `Counter`와 그 state를 제거합니다.
이는 같은 위치에 다른 컴포넌트를 랜더링할 때 컴포넌트는 그 전체 서브 트리의 state를 초기화한다는 것을 알 수 있습니다. 예를 들어 위 코드에서 `Counter`컴포넌트는 유지하되 `<div>`태그를 `<section>`으로 바꿔도 `Counter`컴포넌트는 초기화됩니다.

## 같은 위치에 state를 초기화하기

기본적으로 React는 컴포넌트가 같은 위치를 유지하면 state를 유지합니다.
하지만 가끔 컴포넌트 state를 초기화하고 싶을 때가 있습니다. 다음 예제는 두 선수가 턴 마다 자신의 점수를 추적하는 앱입니다.
```jsx
export default function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  return (
    <div>
      {isPlayerA ? (
        <Counter person="Taylor" />
      ) : (
        <Counter person="Sarah" />
      )}
      <button onClick={() => {
        setIsPlayerA(!isPlayerA);
      }}>
        Next player!
      </button>
    </div>
  );
}

function Counter({ person }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{person}'s score: {score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Add one
      </button>
    </div>
  );
}
```
이 예제는 지금은 점수가 유지 됩니다. 두 `Counter`가 같은 위치에 나타내기 때문에 React는 `person`props가 변경된 같은 `Counter`로 보고있습니다.
하지만 우리가 원하는건 Taylor와 Sarah 두개의 카운터입니다.

이 둘을 바꿀 때 state를 초기화하기 위한 방법은 두 가지가 있습니다.

## 1. 다른 위치에 컴포넌트 랜더링하기

두 `Counter`가 독립적이기를 원한다면 둘을 다른 위치에 랜더링 할 수 있습니다.
```jsx
export default function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  return (
    <div>
      {isPlayerA &&
        <Counter person="Taylor" />
      }
      {!isPlayerA &&
        <Counter person="Sarah" />
      }
      <button onClick={() => {
        setIsPlayerA(!isPlayerA);
      }}>
        Next player!
      </button>
    </div>
  );
}
```
- 처음에는 `isPlayerA`가 `true`입니다. 첫 번째 자리에 `Counter`가 있고 두 번째 자리는 비어있습니다.
- Next plyer를 클릭하면 첫번째 자리는 비워지고 두번째 자리에 `Counter`가 옵니다.
이 방법은 적은 수의 독립된 컴포넌트만을 가지고 있을 때 편리합니다.

## 2. key를 이용해 state를 초기화하기
State를 초기화하는 좀 더 일반적인 방법입니다.

key는 배열을 위한 것만이 아닙니다. React가 컴포넌트를 구별할 수 있도록 key를 사용할 수 있습니다.
기본적으로 React는 컴포넌트를 구별하기 위해 부모 안에서의 순서('첫 번째 카운터', '두 번째 카운터')를 이용합니다. 하지만 key를 이용하면 React에게 특정한 카운터라고 말해줄 수 있습니다.

다음 예시의 두 `<Counter/>`는 JSX에서 같은 위치에 나타나지만, state를 공유하지 않습니다.

```jsx
import { useState } from 'react';

export default function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  return (
    <div>
      {isPlayerA ? (
        <Counter key="Taylor" person="Taylor" />
      ) : (
        <Counter key="Sarah" person="Sarah" />
      )}
      <button onClick={() => {
        setIsPlayerA(!isPlayerA);
      }}>
        Next player!
      </button>
    </div>
  );
}
```
Taylor와 Sarah를 바꾸지만, 다른 `key`를 주었기 때문에 state를 유지하지 않습니다.

`key`를 명시하면 React는 부모 내에서 순서 대신 `key` 자체를 위치의 일부로 사용합니다. 이것이 JSX에서 같은 자리에 랜더링하지만 React 관점에서는 다른 카운터인 이유입니다. 하지만 key가 전역으로 유일하지는 않습니다. key는 오직 부모 안에서만 자리를 명시합니다.

## key를 이용해 폼 초기화하기

key로 state를 초기화하는 것은 특히 폼을 다룰 때 유용합니다.
```jsx
export default function Messenger() {
  const [to, setTo] = useState(contacts[0]);
  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedContact={to}
        onSelect={contact => setTo(contact)}
      />
      <Chat contact={to} />
    </div>
  )
}

const contacts = [
  { id: 0, name: 'Taylor', email: 'taylor@mail.com' },
  { id: 1, name: 'Alice', email: 'alice@mail.com' },
  { id: 2, name: 'Bob', email: 'bob@mail.com' }
];
```
이 채팅 앱에서 `<Chat>`컴포넌트는 문자열 입력 state를 가지고 있습니다.

이 앱에서는 입력란에 타이핑한 후 다른 버튼을 눌러 수신자를 변경하더라도 `<Chat>`가 트리의 같은 곳에서 랜더링 되기 때문에 입력값이 유지됩니다.

다른 앱에서는 필요한 기능이겠지만 채팅 앱이라면 다릅니다. 사용자가 입력한 내용이 실수로 잘못된 사람에게 보내는 것은 바람직하지 않습니다. 이것을 고치기 위해 `key`를 추가해봅시다.

```jsx
export default function Messenger() {
  const [to, setTo] = useState(contacts[0]);
  // key= {to.id} 추가!
  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedContact={to}
        onSelect={contact => setTo(contact)}
      />
      <Chat key={to.id} contact={to} /> 
    </div>
  )
}

const contacts = [
  { id: 0, name: 'Taylor', email: 'taylor@mail.com' },
  { id: 1, name: 'Alice', email: 'alice@mail.com' },
  { id: 2, name: 'Bob', email: 'bob@mail.com' }
];
```
이제 다른 수신자를 선택할 때 `Chat`컴포넌트가 그 트리에 있는 모든 state를 포함해서 처음부터 다시 생성되는 것을 보장합니다. React는 DOM엘리먼트도 다시 사용하는 대신 새로 만들것입니다.