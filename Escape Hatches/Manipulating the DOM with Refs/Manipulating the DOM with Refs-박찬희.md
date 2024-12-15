# Ref로 DOM 조작하기

React는 랜더링 결과물에 맞춰 DOM 변경을 자동으로 처리하기 때문에 컴포넌트에서 자주 DOM을 조작해야 할 필요는 없습니다. 하지만 React가 관리하는 DOM 요소에 접근해야 할 때가 있습니다. React는 이런 작업을 수행하는 내장 방법을 제공하지 않기 때문에 DOM노드에 접근하기 위한 ref가 필요합니다.

## ref로 노드 가져오기

React가 관리하는 DOM 노드에 접근하기 위해 `useRef`Hook이 가져오고 컴포넌트 안에서 ref를 선언합니다.
마지막으로 ref를 DOM 노드로 가져와야하는 JSX tag에 `ref` 어트리뷰트로 전달합니다.
```jsx
import { useRef } from 'react';

const myRef = useRef(null);

<div ref ={myRef}><div> 
// 브라우저 API 사용하기
myRef.current.scrollIntoView();

```

`useRef` Hook은 `current`라는 단일 속성을 가진 객체를 반환합니다. 초기에는 'myRef.current'가 'null'이 됩니다. React가 이 `<div>`에 대한 DOM 노드를 생성할 때, React 는 이 노드에 대한 참조를 `myRef.current`에 넣습니다. 그리고 이 DOM 노드를 이벤트 핸들러에서 접근하거나 정의된 내장 브라우저 API를 사용할 수 있습니다.

## 예시: 텍스트 입력에 포커스 이동하기
아래 예시는 버튼을 클릭하면 input 요소로 포커스를 이동합니다.
```jsx
import { useRef } from 'react';

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <input ref={inputRef} />
      <button onClick={handleClick}>
        Focus the input
      </button>
    </>
  );
}
```

예시 구현을 아래와 같이 해봅시다.
1. `uesRef` Hook을 사용하여 `inputRef`를 선언합니다.
2. 선언한 `inputRef`를 `<input ref={inputRef}>`이와 같이 전달합니다. 해당 코드는 React에 `<input>`의 DOM 노드를 `inputRef.current`에 넣어줘 라고 하는것입니다.
3. `handleClick`함수에서 `inputRef.current`에서 input DOM 노드를 읽고 `inputRef.current.focus()`로 `focus()`를 호출합니다.
4. `<button>`의 `onClick`으로 `handelClick`이벤트 핸들러를 전달합니다.

DOM 조작이 ref를 사용하는 가장 일반적인 사용처지만 `useRef` Hook은 setTimeout Timer ID 같은 React 외부 요소를 저장하는 용도로도 사용할 수 있습니다. 이때 state와 비슷하게 ref는 랜더링 사이에도 유지됩니다. ref를 설정하더라도 컴포넌트의 랜더링을 다시 유발하지 않는 state와 유사합니다.

또한 한 컴포넌트에서 하나 이상의 ref를 가질 수 있습니다.

### ref 콜백을 사용하여 ref 리스트 관리하기

ref는 여러개를 가질 수 있고, 미리 정해진 숫자만큼의 ref뿐만 아니라 ref가 얼마나 많이 필요할지 예측할 수 없는 경우도 있습니다. 이때 아래의 코드는 작동하지 않습니다.
```jsx
<ul>
  {items.map((item) => {
    // 작동하지 않습니다!
    const ref = useRef(null);
    return <li ref={ref} />;
  })}
</ul>
```
왜냐하면 Hook은 컴포넌트의 최상단에서만 호출되어야 하기 때문입니다. `useRef`를 반복문, 조건문 혹은 `map()`메소드 안쪽에서 호출할 수 없습니다.

이 문제를 해결하기 위해서는 부모 요소에서 단일 ref를 얻고, `querySelectorAll`과 같은 DOM조작 메소드를 사용하여 그 안에서 개별 자식 노드를 찾는 것입니다. 하지만 이는 다루기 힘들며 DOM 구조가 바뀌는 경우 작동하지 않을 수 있습니다.

또 다른 해결책은 `ref` 어트리뷰트에 함수를 전달하는 것입니다. 이것을 `ref` 콜백이라고 합니다. React는 ref를 설정할 때 DOM 노드와 함께 ref 콜백을 호출하며, ref를 지울 때에는 null을 전달합니다. 이를 통해 자체 배열이나 Map을 쥬이하고, 인덱스나 특정 ID를 사용하여 어떤 ref에든 접근할수 있습니다.

## 다른 컴포넌트의 DOM 노드 접근하기

`<input/>`같은 브라우저 요소를 출력하는 내장 컴포넌트에 ref를 주입할 때 React는 ref의 `current`프로퍼티를 그에 해당하는 브라우저의 실제 `<input/>` 같은 DOM노드로 설정합니다.

하지만 `<MyInput/>` 같이 커스텀 컴포넌트에 ref를 주입할 때는 `null`이 기본적으로 주어집니다. 다음 예시에서 알아볼 수 있습니다. 이 예시는 버튼을 클릭해도 input 요소에 포커스 되지 않습니다.
```jsx
import { useRef } from 'react';

function MyInput(props) {
  return <input {...props} />;
}

export default function MyForm() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>
        Focus the input
      </button>
    </>
  );
}
```
React는 문제를 인지 할 수 있도록 콘솔에 오류 메시지를 출력합니다.
>Warning: Function components cannot be given refs. Attempts to access this ref will fail. Did you mean to use React.forwardRef()?

React는 기본적으로 다른 컴포넌트의 DOM 노드에 접근하는 것을 허용하지 않습니다. 컴포넌트의 자식도 마찬가지입니다. Ref는 자제해서 사용해야하는 탈출구입니다. 직접 다른 컴포넌트의 DOM 노드를 조작하는 것은 코드가 쉽게 깨지게 만듭니다.

대신 특정 컴포넌트에서 소유한 DOM 노드를 선택적으로 노출할 수 있습니다. 컴포넌트는 자식 중 하나의 ref를 전달하도록 지정할 수 있습니다. 다음 `MyInput`이 `forwardRef` API를 사용하는 것을 보시죠
```jsx
 const MyInput = forwardRef((props, ref) => {
  return <input {...props} ref={ref} />;
});
```

1. `<MyInput ref={inputRef} />`으로 React가 대응하는 DOM 노드를 `inputRef.current`에 대입하도록 설정합니다. 하지만 이것은 `MyInput` 컴포넌트의 선택에 달려있습니다. 기본적으로는 이렇게 동작하지 않습니다.
2. `MyInput`컴포넌트는 `forwardRef`통해 선언 되었고, 이것은 `props` 다음에 선언된 두번째 ref 인수를 통해 상위의 `inputRef`를 받을 수 있도록 합니다.
3. `MyInput`은 자체적으로 수신받은 `ref`를 컴포넌트 내부의 `<input>`으로 전달합니다.

수정한 예제를 보면 이제 버튼을 클릭할때 포커스가 잘 이동합니다.
```jsx
import { forwardRef, useRef } from 'react';

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
      <button onClick={handleClick}>
        Focus the input
      </button>
    </>
  );
}
```

이 패턴은 디자인 시스템에서 버튼, 입력 요소 등의 저수준 컴포넌트에서 DOM 노드를 전달하기 위해 흔하게 사용됩니다. 반면 폼, 목록 혹은 페이지 섹션 등의 고수준 컴포넌트에서는 의도하지 않은 DOM 구조 의존성 문제를 피하고자 일반적으로 DOM 노드를 노출하지 않습니다.

### 명령형 처리방식으로 하위 API 노출하기
위 예시의 DOM 입력 요소를 그대로 노출한것에 반해 노출된 기능을 제한하고 싶을 때는 `useImperativeHandle`을 사용합니다.

```jsx
import {
  forwardRef,
  useRef,
  useImperativeHandle
} from 'react';

const MyInput = forwardRef((props, ref) => {
  const realInputRef = useRef(null);
  useImperativeHandle(ref, () => ({
    // 오직 focus만 노출합니다.
    focus() {
      realInputRef.current.focus();
    },
  }));
  return <input {...props} ref={realInputRef} />;
});

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>
        Focus the input
      </button>
    </>
  );
}
```
여기 `MyInput`내부의 `realInputRef`는 실제 input DOM 노드를 가지고 있습니다. 하지만 `useImperativeHandle`을 사용하여 React가 ref를 참조하는 부모 컴포넌트에 직접 구성한 객체를 전달하도록 지시합니다. 따라서 `Form` 컴포넌트 안쪽의 `inputRef.current`는 `focus`메소드만 가지고 있습니다. 이 경우 ref는 DOM노드가 아니라 `useImperativeHandle`호출에서 직접 구성한 객체가 됩니다.

## React가 ref를 부여할 때
React의 모든 갱신은 두 단계로 나눌 수 있습니다.
- 랜더링 단계에서 React는 화면에 무엇을 그려야하는지 알아내도록 컴포넌트를 호출합니다.
- 커밋 단계에서 React는 변경사항을 DOM에 적용합니다.

일반적으로 랜더링하는 중 ref에 접근하는 걸을 원하지 않습니다. DOM 노드를 보유하는 ref도 마찬가지입니다. 
첫 랜더링에서 DOM 노드는 아직 생성되지 않아서 `ref.current`는 `null`인 상태입니다.
그리고 갱신에 의한 랜더링에서 DOM 노드는 아직 업데이트되지 않은 상태입니다. 두 상황 모두 ref를 읽기에 너무 이른 상황입니다.

React는 `ref.current`를 커밋 단계에서 설정합니다. 
DOM을 변경하기 전에 React는 관련된 `ref.current`값을 미리 `null`로 설정합니다. 
그리고 DOM을 변경한 후 React는 즉시 대응하는 DOM 노드로 다시 설정합니다.

대부분 `ref`접근은 이벤트 핸들러 안에서 일어납니다. ref를 사용하여 뭔가를 하고 싶지만, 그것을 시행할 특정 이벤트가 없을 때 Effect가 필요할 수도 있습니다.

## ref로 DOM을 조작하는 모범 사례

Ref는 탈출구입니다. React에서 벗어나야 할 때만 사용해야 합니다. 포커스 혹은 스크롤 위치에서 관리하거나, React가 노출하지 않는 브라우저 API를 호출하는 등의 작업이 이에 포함됩니다.

포커스 및 스크롤 관리 같은 비 파괴적인 행동을 고수한다면 어떤 문제도 마주치지 않을것입니다.
하지만 DOM을 직접 수정하는 시도를 한다면 React가 만들어내는 변경 사항과 충돌을 발생시킬 위험을 감수해야합니다.

이 문제를 이해하기 위해 이번 예시에서는 환영 문구와 두 버튼을 포함하고 있습니다.
첫 버튼은 일반적인 React 조건부 랜더링과 state를 사용하여 노드 존재 여부를 토글합니다.
두 번째 버튼은 DOM API의 `remove()`를 사용하여 React의 제어 밖에서 노드를 강제적으로 삭제합니다.

```jsx
import {useState, useRef} from 'react';

export default function Counter() {
  const [show, setShow] = useState(true);
  const ref = useRef(null);

  return (
    <div>
      <button
        onClick={() => {
          setShow(!show);
        }}>
        Toggle with setState
      </button>
      <button
        onClick={() => {
          ref.current.remove();
        }}>
        Remove from the DOM
      </button>
      {show && <p ref={ref}>Hello world</p>}
    </div>
  );
}
```

DOM요소를 직접 삭제한 뒤 `setState`를 사용하여 다시 DOM 노드를 노출하는 것은 충돌을 발생시킵니다. 
DOM을 직접 변경했을 때 React는 DOM노드를 올바르게 계속 관리할 방법을 모르기 때문입니다.

React가 관리하는 DOM 노드를 직접 바꾸려하지 마세요. 
React가 관리하는 DOM요소에 대한 수정, 자식 추가 혹은 자식 삭제는 비일관적인 시각적 결과 혹은 위 예시처럼 충돌로 이어집니다.

하지만 항상 이것을 할 수 없다는 의미는 아닙니다.
안전하게 React가 업데이트할 이유가 없는 DOM 노두 일부를 수정할 수 있습니다.
예를 들어 몇몇 `<div>`가 항상 빈채로 JSX에 있다면, React는 해당 노드의 자식 요소를 건드릴 이유가 없습니다.
따라서 빈 노드에서 엘리먼트를 추가하거나 삭제하는 것은 안전합니다.