# Preserving and Resetting State

각 컴포넌트는 독립된 state를 가집니다.

리렌더링마다 언제 state를 보존하고 또 state를 초기화할지 컨트롤할 수 있습니다

학습 내용
React가 언제 state를 보존하고 언제 초기화하는지
어떻게 React가 컴포넌트의 state를 초기화하도록 강제할 수 있는지
key와 타입이 state 보존에 어떻게 영향을 주는지

## 같은 위치에서 state 초기화하기
1. 다른 위치에 컴포넌트를 렌더링하기
2. 각 컴포넌트에 key로 명시적인 식별자를 제공하기


### 1. 다른 위치에 컴포넌트를 렌더링하기
React는 DOM의 동일한 위치에 렌더링된 컴포넌트를 동일한 것으로 간주합니다. 
따라서, 같은 위치에서 상태를 초기화하려면 위치를 변경해야 합니다.

``` jsx
import { useState } from 'react';

export default function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);

  return (
    <div>
      {isPlayerA && <Counter person="Taylor" />}
      {!isPlayerA && <Counter person="Sarah" />}
      <button onClick={() => setIsPlayerA(!isPlayerA)}>
        Next player!
      </button>
    </div>
  );
}

function Counter({ person }) {
  const [score, setScore] = useState(0);

  return (
    <div>
      <h1>{person}'s score: {score}</h1>
      <button onClick={() => setScore(score + 1)}>Add one</button>
    </div>
  );
}

```
isPlayerA가 true면 Taylor의 Counter가 첫 번째 자리에 렌더링됩니다.
버튼을 누르면 첫 번째 자리의 Counter가 제거되고 두 번째 자리에 Sarah의 Counter가 렌더링됩니다.
> **결과: React는 컴포넌트 위치가 달라졌다**고 판단하여 이전의 state를 제거하고 새로운 Counter를 생성합니다.


### 2. key 속성을 사용하여 state 초기화하기

React는 동일한 위치의 컴포넌트를 **부모 내 순서**로 구별합니다. 
key 속성을 사용하면 React가 특정 컴포넌트로 구별하도록 만들 수 있습니다.

``` jsx

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
      <button onClick={() => setIsPlayerA(!isPlayerA)}>
        Next player!
      </button>
    </div>
  );
}

function Counter({ person }) {
  const [score, setScore] = useState(0);

  return (
    <div>
      <h1>{person}'s score: {score}</h1>
      <button onClick={() => setScore(score + 1)}>Add one</button>
    </div>
  );
}
```

Taylor와 Sarah의 Counter는 같은 위치에 렌더링되지만, 
key 속성으로 React가 이 **두 컴포넌트를 별개로 구별**합니다.

React는 key 속성이 다른 경우 이전 state를 삭제하고 새로운 컴포넌트를 생성합니다.

> 결과: key="Taylor"와 key="Sarah"를 사용하여 각각의 상태를 독립적으로 유지합니다. 
상태는 항상 새로운 값으로 초기화됩니다.

## 폼에서 key 속성 활용하기

채팅 앱에서는 메시지 입력 필드가 선택된 수신자에 따라 초기화되어야 합니다. 

하지만 state가 유지되어 사용자가 이전 메시지를 잘못된 사람에게 보낼 위험이 있습니다.

``` jsx
import { useState } from 'react';
import Chat from './Chat.js';
import ContactList from './ContactList.js';

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
  );
}

const contacts = [
  { id: 0, name: 'Taylor', email: 'taylor@mail.com' },
  { id: 1, name: 'Alice', email: 'alice@mail.com' },
  { id: 2, name: 'Bob', email: 'bob@mail.com' }
];
```
 Chat 컴포넌트가 같은 위치에 렌더링되므로, 입력 값이 유지됩니다.

**해결책: key 속성 추가**

``` jsx
<Chat key={to.id} contact={to} />
```

수신자가 변경되면 key 값도 변경됩니다.

React는 기존 컴포넌트를 제거하고 새로운 상태로 초기화된 컴포넌트를 생성합니다.

```
위치 변경: 같은 컴포넌트를 다른 위치에 렌더링하여 state를 초기화.
key 사용: key 속성을 설정하여 같은 위치에 있어도 다른 컴포넌트로 구별.
```