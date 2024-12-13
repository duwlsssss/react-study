# Ref로 값 참조하기

컴포넌트가 일부 정보를 기억하고 싶지만, 해당 정보가 랜더링을 유발하지 않도록 하려면 ref를 사용해야합니다.

## 컴포넌트에 ref 추가

React에서 `useRef` Hook을 가져와 컴포넌트에 ref를 추가할 수 있습니다.
컴포넌트 내에서 `useRef` Hook을 호출하고 참조할 초깃값을 유일한 인자로 잔달합니다.

```jsx
import { useRef } from 'react';

const ref = useRef(0); //0을 초기값으로 설정

//uesRef가 반환하는 객체
{
  current: 0 // uesRef에 전달한 값
}
```
`ref.current`프로퍼티를 통해 해당 ref의 current 값에 접근할 수 있습니다. 이 값은 의도적으로 변경할 수 있어 읽고 쓸 수 있습니다. React가 추적하지 않는 구성 요소의 비밀 주머니라 할 수 있습니다.
다음 예제는 버튼을 클릭할 때 마다 `ref.current`를 증가시킵니다.

```jsx
import { useRef } from 'react';

export default function Counter() {
  let ref = useRef(0);

  function handleClick() {
    ref.current = ref.current + 1;
    alert('You clicked ' + ref.current + ' times!');
  }

  return (
    <button onClick={handleClick}>
      Click me!
    </button>
  );
}
```
예제에서는 ref가 숫자를 가리키지만, state처럼 문자열, 객체, 함수 등 모든 것을 가리킬 수 있습니다. state와 달리 ref는 읽고 수정할 수 있는 `current` 프로퍼티를 가진 일반 자바스크립트 객체입니다.

**컴포넌트는 모든 증가에 대해 다시 랜더링 되지 않습니다.** state와 마찬가지로 ref도 React에 리랜더에 의해 유지됩니다. 하지만 state와 달리 ref는 변경하면 다시 랜더링 되지 않습니다.

## 예시: 스톱워치 작성하기

ref와 state를 단일 컴포넌트로 결합 할 수 있습니다. 예를 들어 사용자가 버튼을 눌러 시작하거나 중지할 수 있는 스톱워치를 만들어봅시다. 사용자가 "start"버튼을 누른 후 시간이 얼마나 지났는지 표시하려면 버튼을 누른 시기와 현재 시각을 추적해야 합니다. 이 정보는 랜더링에 사용되므로 state를 유지합니다.
```jsx
import { useState } from 'react';

export default function Stopwatch() {
  const [startTime, setStartTime] = useState(null);
  const [now, setNow] = useState(null);
  const intervalRef = useRef(null);

  function handleStart() {
    // 카운팅을 시작합니다.
    setStartTime(Date.now());
    setNow(Date.now());

    clearInterval(intervalRef.current);
    intervalRef.current = setInterval(() => {
      //10ms 마다 현재 시간을 업데이트합니다.
      setNow(Date.now());
    }, 10);
  }

  function handleStop() {
    clearInterval(intervalRef.current);
  }

  let secondsPassed = 0;
  if (startTime != null && now != null) {
    secondsPassed = (now - startTime) / 1000;
  }

  return (
    <>
      <h1>Time passed: {secondsPassed.toFixed(3)}</h1>
      <button onClick={handleStart}>
        Start
      </button>
      <button onClick={handleStop}>
        Stop
      </button>
    </>
  );
}
```
시작 버튼을 누르면 setInterval을 사용하여 10ms마다 시간을 업데이트 하고 Stop 버튼을 누르면 `now` state 변수의 업데이트를 중지하기 위해 기존 interval을 취소해야 합니다. 이를 위해 `clearInterval`을 호출하면 됩니다. 그러나 이전에 사용자가 시작을 눌렀을 때 `setInterval` 호출로 반환된 interval ID를 제공해야합니다. interval ID는 어딘가에 보관해야 합니다. 이때  interval ID는 랜더링에 사용되지 않으므로 ref에 저장할 수 있습니다.
랜더링에 정보를 사용할 때 해당 정보를 state로 유지합니다. 이벤트 핸들러에게만 필요한 정보이고 변경이 일어날 때 리랜더링이 필요하지 않다면, ref를 사용하는 것이 더 효율적일 수 있습니다.

## ref와 state의 차이

ref가 state보다 덜 엄격한 것으로 생각될 수 있습니다. 항상 state 설정 함수를 사용하지 않고 변경할 수 있지만 대부분은 state를 사용하고 싶을 것입니다. ref는 자주 필요하지 않은 탈출구입니다.

| refs | state|
| --- | ---|
|`useRef(initialValue)`는 `{current: initialValue}`를 반환합니다.| `useState(initialValue)`는 state 변수의 현재 값과 setter 함수 `[value, setValue]`를 반환합니다.|
|state를 바꿔도 리랜더 되지 않습니다.| state를 바꾸면 리랜더 됩니다.|
|Mutable-랜더링 프로레스 외부에서 `current`값을 수정 및 업데이트 할 수 있습니다.| "Immutable"-state를 수정하기 위해서는 state 설정 함수를 반드시 사용하여 리랜더 대기열에 넣어야 합니다|
|랜더링 중에는 `current`값을 읽거나 쓰면 안됩니다.| 언제든지 state를 읽을 수 있습니다. 그러나 각 랜더마다 변경되지 않는 자체적인 state의 snapshot이 있습니다.|

다음 예제는 state와 함께 구현되는 카운터 버튼입니다.
```jsx
import { useState } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <button onClick={handleClick}>
      You clicked {count} times
    </button>
  );
}
```

`count`값이 표시되므로 state 값을 사용하는 것이 타당합니다. 카운터의 값이 `setCount()`로 설정되면 React는 컴포넌트를 다시 랜더링하고 새 카운트를 반영하도록 화면이 업데이트됩니다.

이를 ref와 함께 구현하려고하면 React는 컴포넌트를 다시 랜더링하지 않으므로 카운트가 변경되는 것을 볼 수 없습니다. 이 버튼을 클릭해도 텍스트가 업데이트되지 않는 방법을 확인해보겠습니다.

```jsx

import { useRef } from 'react';

export default function Counter() {
  let countRef = useRef(0);

  function handleClick() {
    // 이것은 컴포넌트의 리렌더를 일으키지 않습니다!
    countRef.current = countRef.current + 1;
  }

  return (
    <button onClick={handleClick}>
      You clicked {countRef.current} times
    </button>
  );
}
```
랜더링 중에 `ref.current`를 출력하면 신뢰할 수 없는 코드가 나오는 이유입니다. 이 부분이 필요하면 state 대신 사용해야합니다.

### `useRef`의 내부적 동작

`useState`와 `useRef` 모두 React에 의해 제공되지만 `useRef`는 `useState`로 구현할 수 있습니다. 다음은 `useRef`가 구현되는 것을 상상해본 것입니다.

```jsx
// React 내부
function useRef(initialValue) {
  const [ref, unused] = useState({ current: initialValue });
  return ref;
}
```
첫번째 랜더 중에 `useRef`는 `{current: initialValue}`를 반환합니다. 이 객체는 React에 의해 저장되므로 다음 랜더 중에 같은 객체가 반환됩니다. 마치 `useState`의 `setState`함수가 없는 것과 같습니다. `useRef`는 항상 동일한 객체를 반환해야 하니까요!

React는 `useRef`가 실제로 충분히 일반적이기 때문에 built-in 버전을 제공합니다. setter가 없는 일반적인 state 변수라고 생각할 수 있습니다. 객체 지향 프로그래밍에 익숙하다면 refs는 인스턴스 필드를 상기시킬수 있지만 `this.something`대신 `somethingRef.current`처럼 써야합니다.

## refs를 사용할 시기

일반적으로 컴포넌트가 React를 외부와 외부API 컴포넌트의 형태에 영향을 미치지 않는 브라우저 API와 통신해야 할 때 ref를 사용합니다.
- timeout IDs를 저장
- DOM 엘리먼트 저장 및 조작
- JSX를 계산하는 데 필요하지 않은 다른 객체 저장

컴포넌트가 일부 값을 저장해야 하지만 랜더링 로직에 영향을 미치지 않는 경우, refs를 선택합니다.

## refs의 좋은 예시
다음 원칙을 따르면 컴포넌트를 보다 쉽게 예측 가능합니다.
- **refs를 탈출구로 간주합니다.** Refs는 외부 시스템이나 브라우저 API로 작업할 때 유용합니다. 앱 로직과 데이터 흐름의 상당 부분이 refs에 의존한다면 접근 방식을 재고하는 것이 좋습니다.

- **랜더링 중에 `ref.current`를 읽거나 쓰지 마세요.** 랜더링 중 일부 정보가 필요한 경우 state를 대신 사용해야합니다. `ref.current`가 언제 변하는지 React는 모르기 때문에 랜더링할 때 읽어도 컴포넌트의 동작을 예측하기 어렵습니다. (`if(!ref.current) ref.current = new Thing()과 같은 코드는 첫 번째 랜더 중에 ref를 한번만 설정하느 경우는 예외입니다.)

React state의 제한은 refs에 적용되지 않습니다. 예를 들어 state는 모든 랜더링에 대한 snapshot 및 동기적으로 업데이트되지 않는 것과 같이 작동합니다. 그러나 ref의 current 값을 변조하면 다음과 같이 즉시 변경됩니다.
```js
ref.current = 5;
console.log(ref.current); //5
```
이는 ref 자체가 일반 자바스크립트 객체처럼 동작하기 때문입니다.
또한 ref로 작업할 때 mutation 방지에 대해 걱정할 필요가 없습니다. 변형하는 객체가 랜더링에 사용되지 않는 한 React는 ref 혹은 해당 콘텐츠를 어떻게 처리하든 신경쓰지 않습니다.

## Refs와 DOM

임의의 값을 ref로 지정할 수 있습니다. 그러나 ref의 가장 일반적인 사용 사례는 DOM 엘리먼트에 엑세스하는 것입니다. 예를 들어 프로그래밍 방식으로 입력의 초점을 맞추려는 경우입니다. `<div ref={myRef}>`와 같은 JSX의 `ref`어트리뷰트에 ref를 전달하려면 React는 해당 DOM엘리먼트를 `myRef.current`에 넣습니다. 만약 엘리먼트가 DOM에서 사라지면, React는 `myRef.current`값을 `null`로 업데이트 합니다.