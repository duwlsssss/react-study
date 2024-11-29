### React에선 State가 불변성을 유지해야한다.
  - state가 렌더링 사이에서 어떻게 바뀌었는지 명확히 알아야 디버깅, 최적화, 캐싱 용이
  - state에 변경사항이 생기면 값이 변해야 의도한대로 리렌더링이 일어남 

### 1. Object State 업데이트 

state는 모든값을 가질 수 있다.
`Primitive value state`는 업데이트시 값 자체가 새 값으로 교체돼 리렌더링을 유발하지만
`Object value state`는 참조값으로 전달되기 때문에 객체 내부를 변경해도 React는 이를 감지하지 못함(React는 참조값의 변화를 감지해 컴포넌트를 리렌더링함)

```jsx
const [person, setPerson] = useState({ name: "John" });

person.name = "Doe"; // 객체 내부 값을 변경
// ☑️ 참조값이 바뀐게 아니니까 컴포넌트 리렌더링 X
console.log(person); // { name: "Doe" }
```

=> State는 불변성을 유지해야 함 === **객체를 state로 사용할 때 절대 원본 객체를 수정하지 말고, state setter 함수에 `객체의 복사본`을 전달해야 함**

```jsx
import { useState } from 'react';

export default function MovingDot() {
  const [position, setPosition] = useState({
    x: 0,
    y: 0
  });
  return ( // setPosition에 새 객체를 넣어 position을 이 객체로 교체하고, 컴포넌트를 리렌더링하라고 요청함
    <div
      onPointerMove={e => {
        setPosition({
          x: e.clientX,
          y: e.clientY
        });
      }}
      ...
    >
    ...
    </div>
  );
}

// 위 코드와 같은 의미임
<div
  onPointerMove={e => {
    const nextPosition = {};
    nextPosition.x = e.clientX;
    nextPosition.y = e.clientY;
    setPosition(nextPosition);
  }}
  ...
>
```

### 2. 중첩 object state 업데이트

객체의 모든 항목의 복사본을 만들어야 함

```jsx
const [person, setPerson] = useState({
  name: 'Niki de Saint Phalle',
  artwork: {
    title: 'Blue Nana',
    city: 'Hamburg',
    image: 'https://i.imgur.com/Sd1AgUOm.jpg',
  }
});

// 여기서 image를 바꾸려면

function handleImageChange(e) {
  setPerson({
    ...person // 동일한 값 사용
    artwork: { // artwork 교체
      ...person.artwork // 동일한 값 사용
      image: e.target.value
    }
  })
}
```

### 3. 깊은 중첩 object state 업데이트

state가 깊이 중첩돼있거나, 복사 코드가 반복되면 `Immer 라이브러리`를 사용하는게 유용함

useState 대신 `useImmer`로 업데이트하고 싶은 부분만 간단히 업데이트 가능

useImmer는 내부적으로 현재 state의 복사본인 `draft`라는 객체를 생성, draft는 Proxy를 통해 상태 변경을 감지하고, 자동으로 새로운 불변 상태를 생성함

useState와 useImmer를 섞어서 사용 가능

```jsx
import { useImmer } from 'use-immer';

const [person, updatePerson] = useImmer({
  name: 'Niki de Saint Phalle',
  artwork: {
    title: 'Blue Nana',
    city: 'Hamburg',
    image: 'https://i.imgur.com/Sd1AgUOm.jpg',
  }
});

function handleImageChange(e) {
  updatePerson(draft => {
    draft.artwork.image = e.target.value;
  });
}
```