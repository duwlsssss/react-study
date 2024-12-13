# Ref로 DOM 조작하기

React는 일반적으로 렌더링 결과물에 맞춰 DOM을 처리. 그러나 포커스, 스크롤 위치바꾸기, 위치와 크기 측정등 상황에서 DOM에 직접 접근이 필요할 때가 있음. 이때 ref를 사용하면 용이함.

```jsx
const myRef = useRef(null);
<div ref={myRef}>
```

jsx태그에 ref 속성을 전달시, current엔 div의 실제 DOM 노드가 저장됩니다. 따라서 직접적으로 DOM요소에 접근할 수 있고, 조작할 수 있습니다.

```jsx
import { useRef } from "react";

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <input ref={inputRef} />
      <button onClick={handleClick}>Focus the input</button>
    </>
  );
}
```

해당 코드의 경우, ref를 통해 input의 실제DOM 요소가 inputRef에 담겨져있습니다. 따라서 버튼을 눌러 handleClick을 호출하게되면, focus()가 바로 켜지게 되는거죠.

## 다른 컴포넌트의 DOM 노드 접근하기

React는 다른 컴포넌트의 DOM 노드에 접근하는 것을 허용하지 않습니다 => 타 컴포넌트에 ref속성 사용 못한다는 얘기죠.

이러한 경우, 애초에 다른 컴포넌트의 인수에 ref를 넣어 상위 컴포넌트의 ref를 받을 수 있도록 할 수 있습니다.

### 다만 이 경우, DOM 구조 의존성 문제가 있으니 폼,목록, 페이지섹션 등엔 사용을 지양해야합니다.

## React가 ref를 부여할 때.

ref는 렌더링 상태때 값을 null로 갖으며, 이후에 DOM을 변경 한 후, 대응하는 DOM 노드로 값을 변경합니다.

=> ref의 접근은 보통 이벤트 핸들러로 일어나며, 이벤트가 없다면 useEffect를 사용하기도 합니다.

## DOM 조작시 주의점

리액트는 DOM노드를 관리하는 방법을 모르기에, DOM노드를 직접 바꾸려하면, 예기치못한 충돌이 발생할 수 있습니다. 따라서 업데이트할 이유가 없는 DOM노드가 아닌이상 직접적인 DOM노트의 조작은 주의해야합니다.
