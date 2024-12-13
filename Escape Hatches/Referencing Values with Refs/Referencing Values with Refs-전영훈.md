# Ref로 값 참조하기

const ref = useRef(0)

이건 왜쓸까여? => 렌더링은 하고싶지 않지만, 어떤 정보를 기억하고 싶을때 사용합니다.

useRef는

```jsx
{
  current: 0;
}
```

이 객체를 반환합니다.

위의 정보와 다시 종합해보자면, Ref라는 박스 내부에 current라는 어떤 것을 저장 가능한 내용물이 있는 셈이죠. 이를 통해 우리는 js에서 흔히 flag로 사용하던 어떤 변수로서 다만 여기선 화면에는 출력되지 않는 변수로서 사용할 수 있게됩니다.

## ref와 state의 차이점

1. ref는 객체를 반환하는 반면, state 변수의 현재 값과 setter 함수 [value, setValue] 를 반환합니다.

2. ref는 리렌더링을 유발하지 않는 반면, state는 값이 바뀌면 리렌더링을 유발합니다.

3. ref의 current는 직접 수정이 가능하지만, state는 항상 set함수를 통해 수정 가능합니다.

## 언제 쓰면 좋을까!

보통은 DOM 조작을 위해 많이 사용합니다.

```jsx
import React, { useRef } from "react";

function TextInputWithFocusButton() {
  const inputEl = useRef(null);
  const onButtonClick = () => {
    // `current`는 마운트된 텍스트 입력 요소를 가리킵니다.
    inputEl.current.focus();
  };
  return (
    <>
      <input ref={inputEl} type="text" />
      <button onClick={onButtonClick}>Focus the input</button>
    </>
  );
}
```
