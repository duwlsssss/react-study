### Ref로 값 참조하기

useState, useEffect와 달리 컴포넌트가 정보를 기억하게 하지만 
렌더링에 사용하진 않고 싶을 때 `useRef` 훅을 사용한다.

`refs`
- React의 렌더링 흐름과 독립적이며, 리렌더링을 트리거하지 않음
- 렌더링 중 current 값을 읽거나 쓰면 안됨 - 리렌더링을 발생시키지 않으니까 예측하지 못한 값 사용 가능, 반면 state는 렌더링 중 읽고, 쓰는 게 안전함
- 렌더링 프로세스 외부에서는 current값을 수정할 수 있음(mutable) - 반연 state를 수정하기 위해선 state setter 함수를 사용해 리렌더링 대기열에 넣어야 함(immutable)
- 사용 사례: 외부 시스템이나 브라우저 API를 사용할때 컴포넌트의 정보 유지 / DOM API 사용해 DOM element 조작할 때 

```js
// 원리
function useRef(initialValue) {
  const [ref, unused] = useState({ current: initialValue });
  return ref;
}
// ref는 { current: initialValue }의 참조값을 가지고 ref.current를 사용해 객체를 직접 조작함
// useState는 객체를 직접 변경하면 리렌더링을 발생시키지 않으니까
// ref.current를 업데이트해도 리렌더링이 일어나지 않음
```

```js
// 사용 예시
import { useRef } from 'react';

export default function Counter() {
  const ref = useRef(0); // 초기값으로 모든 값 설정 가능
 
  function handleClick() {
    ref.current = ref.current + 1;  // ref.current 프로퍼티를 통해 해당 ref의 current 값에 접근 가능
    alert('You clicked ' + ref.current + ' times!');
  }

  return (
    <button onClick={handleClick}>
      Click me!
    </button>
  );
}
```