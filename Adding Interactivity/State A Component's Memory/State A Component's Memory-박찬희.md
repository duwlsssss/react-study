# State A Component's Memory

state: 현재 입력값, 현재 이미지, 장바구니 등 컴포넌트의 상호작용으로 생긴 메모리

## 일반 변수로 충분하지 않은 경우
```jsx
import { sculptureList } from './data.js';

export default function Gallery() {
  let index = 0;

  function handleClick() {
    index = index + 1;
  }

  let sculpture = sculptureList[index];
  return (
    <>
      <button onClick={handleClick}>
        Next
      </button>
      <h2>
        <i>{sculpture.name} </i> 
        by {sculpture.artist}
      </h2>
      <h3>  
        ({index + 1} of {sculptureList.length})
      </h3>
      <img 
        src={sculpture.url} 
        alt={sculpture.alt}
      />
      <p>
        {sculpture.description}
      </p>
    </>
  );
}
```
위 예시에서 `handleClick` 이벤트 핸들러는 `index`를 업데이트 하고 있지만 페이지는 변화가 없습니다.

지역 변수는 랜더링간 유지 되지 않고 변경 되더라도 랜더링을 일으키지 않기 때문입니다.

이걸 해결하기 위해서는 `useState` 훅을 사용합니다.
#### useState
`useState`훅은 두 가지를 제공합니다.
1. 랜더링 간 데이터 유지를 위한 state변수
2. 변수를 업데이트하고 컴포넌트를 다시 랜더링하도록 유발하는 state setter함수

## state 변수 추가하기
state 변수를 추가하기 위해서 React에서 `useState`를 가져옵니다. 
```jsx
import { useState } from 'react';

// let index = 0;
const [index, setIndex] = useState(0); // state로 index 변수 선언 setIndex = setter 함수
```

`index`를 새로 선언하고 `handleClick`을 수정합니다
```jsx
// function handleClick() {
//   index = index + 1;
// }
function handleClick() {
  setIndex(index + 1);
}
```
이 두가지 수정을 거치면 버튼 클릭시 다음 페이지로 전환됩니다.

## 첫 번째 훅 만나기
Hook: React에서 `use`로 시작하는 다른 모든 함수, 오직 랜더링 중일 때만 사용 가능한 함수

> 주의! <br>
Hook은 컴포넌트의 최상위 수준 또는 커스텀 훅에서만 호출할 수 있습니다. 조건문, 반복문 또는 기타 중첩 함수 내부에서는 훅을 호출할 수 없습니다. 훅은 함수이지만 컴포넌트의 필요에 대한 무조건적인 선언으로 생각하면 도움이 됩니다. 파일 상단에서 모듈을 “import”하는 것과 유사하게 컴포넌트 상단에서 React 기능을 “사용”합니다.

## `useState` 해부하기
`useState`는 `const [something, setSomething]`과 같이 지정하는 것이 규칙입니다.

`useState`의 유일한 인수는 state변수의 초깃값입니다.
```jsx
const [index, setIndex] = useState(0);
```
위 예시를 보았을 때 `index`는 변수의 역할을 하며, 초기값은 `useState(0)`에서 인수로 부여한 `0`이 됩니다.

또한 `setIndex`는 함수로 호출하게 되면 인자로 받은 것에 따라 state를 업데이트 하고 컴포넌트를 랜더링합니다. 
`setIndex(index+1)` 이런 식으로 호출 하면 index가 1인 것을 기억하며 다시 랜더링되고 `[1, setIndex]`를 반환하며 이런식으로 반복됩니다.

## 컴포넌트에 여러 state 변수 지정하기
하나의 컴포넌트에 원하는 만큼 많은 타입의 state 변수를 가질 수 있습니다. 단, `useState`는 최상위 수준에서만 훅을 호출해야 합니다.

## State는 격리되고 비공개로 유지됩니다
State는 컴포넌트 인스턴스에서 지역적입니다. 동일한 컴포넌트라도 독립된 state를 가지며 하나가 변경 되더라도 다른 컴포넌트에는 영향을 미치지 않습니다.