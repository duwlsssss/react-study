### State를 보존하고 초기화하기

React가 컴포넌트가 같은지 여부를 판단하는 기준:
- 같은 타입의 컴포넌트인가
- `UI 트리`에서 같은 `위치`에 렌더링되는가
- key값이 동일한가? (컴포넌트에 key를 주면 React는 UI트리에서의 위치 대신 key를 위치로 사용함)

이 조건을 만족하면 같은 컴포넌트라 간주하고 state를 유지함, 생명주기가 변경되지 않음

반면 다른 컴포넌트로 판단되면 기존 컴포넌트를 언마운트하며  state가 삭제되고, 새 컴포넌트가 마운트되며 초기 state가 설정됨

```jsx
import { useState } from 'react';

export default function App() {
  const [isFancy, setIsFancy] = useState(false);
  // 이 경우 같은 위치(div의 첫번째 자식)에 Counter가 렌더링되므로 state가 유지됨, 위치가 달라지면 state가 초기화됨
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

=> 리렌더링시 state를 유지하고 싶으면 트리구조를 완전히 같게 유지해야함

<br/> 

➕ 같은 위치에 컴포넌트를 렌더링하며 state를 초기화하고 싶을 수도 있음

```jsx
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
      <Chat key={to.id} contact={to} />
      {/* ✅ 키값이 바뀌면 위치가 바뀐다고 간주해 그 컴포넌트 포함, 하위 트리의 state가 모두 초기화됨 */}
    </div>
  )
}

const contacts = [
  { id: 0, name: 'Taylor', email: 'taylor@mail.com' },
  { id: 1, name: 'Alice', email: 'alice@mail.com' },
  { id: 2, name: 'Bob', email: 'bob@mail.com' }
];
```


