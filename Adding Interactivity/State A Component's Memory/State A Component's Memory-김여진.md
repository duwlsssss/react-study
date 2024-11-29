### State: 컴포넌트의 저장소

상호작용의 결과로 데이터가 변경되며 컴포넌트 재렌더링이 일어나야 할때 데이터가 유지돼야함(컴포넌트가 데이터를 기억해야함)

`useState hook`은 
데이터의 초기값을 유일한 매개변수로 입력받아
렌더링간에 데이터를 유지하기 위한 state변수,
변수를 업데이트하고 컴포넌트를 재렌더링하도록 하는 state setter함수로 이뤄진 배열 반환함

컴포넌트의 최상위 수준에서만 호출해야함

state는 각 컴포넌트에 지역적임, 같은 컴포넌트여도 독립됨 state를 가지고, 다른 컴포넌트에 영향을 주지 않음

```jsx
import { useState } from 'react';
import { sculptureList } from './data.js';

export default function Gallery(){
  const [index, setIndex] = useState(0); // index는 state 변수, setIndex 는 setter 함수

  function handleClick() {
    setIndex(index + 1); // 버튼을 클릭하면 setIndex(index + 1)를 호출 -> React에 index는 index+1임을 기억하게 하며 또 다른 렌더링을 유발
  }

  let sculpture = sculptureList[index];
  return (
    <>
      <button onClick={handleClick}>Next</button>
      <h2>{sculpture.name}</h2>
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

