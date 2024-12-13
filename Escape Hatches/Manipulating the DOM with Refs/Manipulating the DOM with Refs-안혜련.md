## ref란 무엇인가?
- React의 useRef Hook은 React가 관리하는 DOM 요소나 값에 직접 접근할 수 있는 도구입니다.
- DOM 요소에 접근하여 포커스 설정, 스크롤 이동, 크기 및 위치 측정 등의 작업을 수행.
- React 외부의 값(예: 타이머 ID)을 저장.
## ref로 DOM 요소 접근하기
1. useRef로 ref 생성:

``` javascript

import { useRef } from 'react';

const myRef = useRef(null); // 초기값은 null
```
2. DOM 요소에 ref 연결:

``` javascript

<div ref={myRef}></div>
```
3. ref.current로 DOM 요소 사용:

- React가 해당 DOM 요소를 생성한 후, myRef.current에 DOM 노드가 저장됩니다.
- 브라우저 API를 사용할 수 있습니다:
``` javascript

myRef.current.focus();
myRef.current.scrollIntoView({ behavior: 'smooth' });
```
###  예제: 입력 필드에 포커스
``` javascript

import { useRef } from 'react';

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus(); // input 요소에 포커스
  }

  return (
    <>
      <input ref={inputRef} />
      <button onClick={handleClick}>Focus the input</button>
    </>
  );
}
```
### 여러 개의 ref 사용
- 여러 DOM 요소에 각각 ref를 설정할 수 있습니다.

예제: 스크롤 이동
``` javascript

import { useRef } from 'react';

export default function CatCarousel() {
  const firstCatRef = useRef(null);
  const secondCatRef = useRef(null);
  const thirdCatRef = useRef(null);

  function scrollTo(ref) {
    ref.current.scrollIntoView({ behavior: 'smooth', block: 'nearest' });
  }

  return (
    <>
      <button onClick={() => scrollTo(firstCatRef)}>Scroll to Cat 1</button>
      <button onClick={() => scrollTo(secondCatRef)}>Scroll to Cat 2</button>
      <button onClick={() => scrollTo(thirdCatRef)}>Scroll to Cat 3</button>

      <div>
        <img ref={firstCatRef} src="cat1.jpg" alt="Cat 1" />
        <img ref={secondCatRef} src="cat2.jpg" alt="Cat 2" />
        <img ref={thirdCatRef} src="cat3.jpg" alt="Cat 3" />
      </div>
    </>
  );
}
```
## Ref와 Forwarding
### Forwarding이 필요한 이유
- 기본적으로 React는 커스텀 컴포넌트(예: `<MyInput />`)에 ref를 전달하지 않습니다.
- DOM 노드에 접근해야 할 경우 **React.forwardRef**를 사용해야 합니다.

#### 예제: ForwardRef를 사용한 커스텀 컴포넌트
``` jsx
import { forwardRef, useRef } from 'react';

// MyInput 컴포넌트가 ref를 전달받아 DOM에 연결
const MyInput = forwardRef((props, ref) => {
  return <input {...props} ref={ref} />;
});

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>Focus the input</button>
    </>
  );
}
```

## Ref 사용 시 주의사항
1. React가 DOM을 관리한다는 사실

- React는 DOM 노드의 상태를 추적하며 렌더링과 동기화합니다.
- 직접 DOM을 수정하면 충돌이 발생할 수 있음.
2. 안전한 DOM 조작:

- DOM 노드를 직접 수정해야 한다면, React가 해당 노드나 자식 요소를 변경하지 않을 부분만 수정.
``` javascript

myRef.current.textContent = "Hello"; // 안전한 수정
```

3. React DOM과 직접 DOM 수정 충돌 사례:

``` javascript

import { useState, useRef } from 'react';

export default function App() {
  const [show, setShow] = useState(true);
  const ref = useRef(null);

  return (
    <>
      <button onClick={() => setShow(!show)}>Toggle</button>
      <button onClick={() => ref.current.remove()}>Force Remove</button>
      {show && <div ref={ref}>Hello World</div>}
    </>
  );
}
```
- ref.current.remove()로 강제 삭제하면 React가 이를 추적하지 못해 문제가 발생할 수 있음.
4. 렌더링 중 ref 사용 금지:

- 렌더링 중에는 DOM이 아직 생성되지 않았거나 최신 상태가 아닙니다.


## useRef의 적절한 사용 사례
1. 포커스 설정: 특정 입력 필드로 포커스를 이동.
2. 스크롤 이동: 특정 요소로 부드럽게 스크롤.
3. 크기 측정: DOM 요소의 크기 및 위치 확인.
4. React 외부 값 저장: 타이머 ID 또는 상태와 무관한 값 저장.

ref는 React에서 "탈출구"로 사용됩니다. 필수적인 경우에만 신중히 사용해야 React의 데이터 흐름과 상태 관리를 효율적으로 유지할 수 있습니다.