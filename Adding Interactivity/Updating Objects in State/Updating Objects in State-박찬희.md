# 객체 State 업데이트하기

state는 모든 종류의 값을 저장할 수 있지만 불변성을 가지고 있습니다. react는 state가 변경될 경우에만 리랜더링 되기 때문입니다. 만약 state가 불변하지 않는다면, state 변경의 기준이 콜 스택 주소값인 react는 리랜더링할 수 없을 것입니다.

```jsx
import { useState } from 'react';

export default function MovingDot() {
  const [position, setPosition] = useState({
    x: 0,
    y: 0
  });
  return (
    <div
      onPointerMove={e => {
        position.x = e.clientX;
        position.y = e.clientY;
      }}
      /* onPointerMove={e => {
        setPosition({
        x : e.clientX.
        y : e.clientY
        })
      }}
      */
      style={{
        position: 'relative',
        width: '100vw',
        height: '100vh',
      }}>
        <div style={{
          position: 'absolute',
          backgroundColor: 'red',
          borderRadius: '50%',
          transform: `translate(${position.x}px, ${position.y}px)`,
          left: -10,
          top: -10,
          width: 20,
          height: 20,
        }} />
    </div>
  )
}
```

다음 예제는 뷰포트 영역 내 마우스 포인터를 따라 빨간 점이 이동해야 합니다.
하지만 position 객체 내부의 값을 직접 변경함으로써 state의 불변성은 깨지게 되고 빨간 점의 위치 값은 변경되겠지만 react는 그것을 감지하지 못하고 리랜더링 되지 않습니다. 따라서 주석과 같이 변경하면 정상적으로 작동합니다.

## 전개 문법으로 객체 복사하기
이전 예제는 항상 새로운 값을 가지고 오기 때문에 리랜더링 되지만 다음 예제는 state 객체의 일부 값만 변경하고 있습니다.
```jsx
import { useState } from 'react';

export default function Form() {
  const [person, setPerson] = useState({
    firstName: 'Barbara',
    lastName: 'Hepworth',
    email: 'bhepworth@sculpture.com'
  });

  function handleFirstNameChange(e) {
    person.firstName = e.target.value;
  }
  /* function handleFirstNameChange(e) {
      setPerson({
      ...person,    전개 후 state변경 주의!
      person.firstName : e.target.value
      })
    }  
  */
  function handleLastNameChange(e) {
    person.lastName = e.target.value;
  }
  /* function handleLastNameChange(e) {
      setPerson({
      ...person,    전개 후 state변경 주의!
      person.lastName : e.target.value
      })
    }  
  */
  function handleEmailChange(e) {
    person.email = e.target.value;
  }
  /* function handleEmailChange(e) {
      setPerson({
      ...person,    전개 후 state변경 주의!
      person.email : e.target.value
      })
    }  
  */
  return (
    <>
      <label>
        First name:
        <input
          value={person.firstName}
          onChange={handleFirstNameChange}
        />
      </label>
      <label>
        Last name:
        <input
          value={person.lastName}
          onChange={handleLastNameChange}
        />
      </label>
      <label>
        Email:
        <input
          value={person.email}
          onChange={handleEmailChange}
        />
      </label>
      <p>
        {person.firstName}{' '}
        {person.lastName}{' '}
        ({person.email})
      </p>
    </>
  );
}
```
본 예제 또한 객체 데이터 자체를 변경하고 setState함수를 사용하지 않아 값을 변경하더라도 react는 감지하지 못하고 리랜더링 되지 않습니다.
setState를 사용하더라도 하나의 값을 변경하기 위해 input에 값을 입력하게 된다면 위 함수는 변경된 state를 통해 리랜더링 되지만 나머지 state 값들은 잃어버리게 되어 사라집니다.
혹은 다른 값은 유지 되도록
```jsx
setPerson({
  firstName: e.target.value, // input의 새로운 first name
  lastName: person.lastName,
  email: person.email
});
```
이와 같이 각 함수마다 불필요하게 같은 값들을 적어줘야합니다.
이때 `...` 객체 전개 구문을 사용하여 모든 프로퍼티를 각각 복사하지 않아도 됩니다. 변경된 내용은 주석으로 표시했습니다.

또한 위 예제의 handleChange 함수들을 하나로 묶을 수도 있습니다.
```jsx
  function handleChange(e) {
    setPerson({
      ...person,
      [e.target.name]: e.target.value
    });
  }
  // onChange={handleEmailChange} => onChange={handleChange}
```
위와 같이 `[]`를 통해 동적 이름을 가진 프로퍼티를 명시할 수 있습니다.

## 중첩된 객체 갱신하기

```jsx
const [person, setPerson] = useState({
  name: 'Niki de Saint Phalle',
  artwork: {
    title: 'Blue Nana',
    city: 'Hamburg',
    image: 'https://i.imgur.com/Sd1AgUOm.jpg',
  }
});
```
위 객체에서 `person.artwork.city`를 state의 불변성을 깨지 않고 업데이트 하기 위해서는 아래와 같이 함수를 호출할 수 있습니다.

```jsx
function handleCityChange(e) {
  setPerson({
    ...person, // 다른 필드 복사
    artwork: { // artwork 교체
      ...person.artwork, // 동일한 값 사용
      city: 'New Delhi' // 하지만 New Delhi!
    }
  });
}
```
### Immer로 간결한 갱신 로직 작성하기
Immer는 react에서 쉽게 불변성을 유지할 수 있는 코드 작성을도와주는 라이브러리입니다.
마치 불변성에 신경쓰지 않는 것처럼 코드를 작성할 수 있게 해주되 불변성 관리를 제대로 해주는 것입니다.

바로 위 예제를 Immer를 사용하여 나타내면 다음과 같습니다.
```jsx
import { useImmer } from 'use-immer';

export default function Form() {
  const [person, updatePerson] = useImmer({
    name: 'Niki de Saint Phalle',
    artwork: {
      title: 'Blue Nana',
      city: 'Hamburg',
      image: 'https://i.imgur.com/Sd1AgUOm.jpg',
    }
  });

  function handleCityChange(e) {
    updatePerson(draft => {
      draft.artwork.city = e.target.value;
    });
  }
}
```
Immer에서 제공하는 `draft`는 Proxy라고 하는 객체 타입으로 어느 부분이 변경되었는지 기록하고 알아내어 변경사항을 포함한 새로운 객체를 생성합니다.