# Queueing a Series of State Updates

state 변수를 설정하면 다음 랜더링이 큐에 들어갑니다

## React state batches 업데이트
```jsx
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 1);
        setNumber(number + 1);
        setNumber(number + 1);
      }}>+3</button>
    </>
  )
}
```
`setNumber(number+1)`이 세 번 호출되므로 버튼을 클릭하면 세 번 증가할 것으로 예상할 수 있지만 각 랜더링의 state 값은 고정되어 있기 때문에 `1`이 반환됩니다.

하지만 여기에는 이 말고도 한 가지 이유가 더 있습니다. React는 state를 업데이트 하기 전 이벤트 핸들러의 모든 코드가 실행될 때까지 기다립니다. 이 때문에 리랜더링은 모든 `setNumber()` 호출이 완료 된 후에 일어납니다.

이때문에 너무 많은 리랜더링이 발생하지 않고 다수의 state 변수를 업데이트 할 수 있습니다. batching은 이벤트 핸들러와 그 안에 있는 코드가 완료될 때까지 UI가 업데이트 되지 않는 것을 의미합니다. 이는 앱의 최적화에 도움을 주며 일부 변수만 업데이트 된 혼란스러운 랜더링을 처리하지 않아도 됩니다.

React는 클릭과 같은 의도적인 이벤트에 대해 batch를 수행하지 않으며 각 클릭은 개별적으로 처리됩니다.

## 다음 랜더링 전 동일한 state 변수를 여러번 업데이트하기

위 함수의 의도와 같이 다음 랜더링 전에 동일한 state 변수를 여러 번 업데이트 하고 싶다면 아래와 같이 변경 할 수 있습니다.
```jsx
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(n => n + 1);
        setNumber(n => n + 1);
        setNumber(n => n + 1); // 단순 state 대체가 아닌 state으로 무언가를 하라고 지시하는 방법
      }}>+3</button>
    </>
  )
}
```
`setNumber(n => n + 1)`와 같은 함수를 업데이터 함수라고 부릅니다. 이는 state 설정자 함수에 전달할때 이벤트 핸들러의 다른 코드가 모두 실행 된 후에 이 함수가 처리하도록 큐에 넣고 최종 업데이트 된 state를 제공합니다.

## state를 교체 후 업데이트 하면?

```jsx
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 5);
        setNumber(n => n + 1);
      }}>Increase the number</button>
    </>
  )
}
```
이런 식으로 함수를 교체하게 된다면 어떻게 될까요?

이 이벤트 핸들러가 React에 지시하는 작업은 다음과 같습니다.

`setNumber(number + 5)` : `number`는 `0`이므로 `setNumber(0 + 5)`입니다. React는 큐에 “5로 바꾸기” 를 추가합니다.
`setNumber(n => n + 1)` :` n => n + 1`는 업데이터 함수입니다. React는 해당 함수를 큐에 추가합니다.
다음 렌더링하는 동안 React는 state 큐를 순회합니다.

| queued update	| n	| returns |
|---|---|---|
| ”replace with 5” |	0 (unused) |	5 |
| n => n + 1 |	5	| 5 + 1 = 6 |
React는 `6`을 최종 결과로 저장하고 `useState`에서 반환합니다.

## 업데이트 후 state를 바꾸면?
```jsx
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 5);
        setNumber(n => n + 1);
        setNumber(42);
      }}>Increase the number</button>
    </>
  )
}
```
위의 예제와 같은 경우는 이 전 예제에서 `setNumber(42)`를 통해 `42`로 바꾸기를 큐에 추가하며 최종 결과로 `42`를 저장하고 `useState`에 반환합니다.

`setNumber state` 설정자 함수에 전달할 내용은 다음과 같이 생각할 수 있습니다:

- 업데이터 함수 (예. n => n + 1) 가 큐에 추가됩니다.
- 다른 값 (예. 숫자 5) 은 큐에 “5로 바꾸기”를 추가하며, 이미 큐에 대기 중인 항목은 무시합니다.

## 명명 규칙
업데이터 함수 인수의 이름은 해당 state 변수의 첫 글자로 지정하는 것이 일반적입니다.
```jsx
setEnabled(e => !e);
setLastName(ln => ln.reverse());
setFriendCount(fc => fc * 2);
```
좀 더 자세한 코드를 선호하는 경우 `setEnabled(enabled => !enabled)`와 같이 전체 state 변수 이름을 반복하거나, `setEnabled(prevEnabled => !prevEnabled)`와 같은 접두사를 사용하는 것이 널리 사용되는 규칙입니다.