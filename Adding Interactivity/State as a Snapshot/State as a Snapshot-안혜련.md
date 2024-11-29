# State as a Snapshot
" state가 단순한 변수처럼 보이지만 실제로는 "스냅샷"처럼 동작한다는 것"
> 스냅샷은 React가 렌더링한 그 시점의 UI 상태를 말합니다.

State 변수는 읽고 쓸 수 있는 일반 자바스크립트 변수처럼 보일 수 있다. 
```
let number = 0;
number = 1;
```
이렇게 하면 변수 number의 값이 바로 변경되지만  React의 state는 다르다.

state를 설정하면 현재 값이 바로 바뀌는 게 아니라, 리렌더링을 요청한다.


-  state는 스냅샷처럼 동작한다. 

- state 변수를 설정하여도 이미 가지고 있는 state 변수는 변경되지 않고, 대신 리렌더링이 일어난다.


## **렌더링**


- 이 스냅샷은 특정 시점의 `state`, `props`, 로컬 변수로 계산된 결과

- 렌더링 이후, React는 UI를 업데이트하고 이벤트 핸들러를 연결한다.

### React가 컴포넌트를 다시 렌더링할 때는

1. React가 함수를 다시 호출
2. 함수가 새로운 JSX 스냅샷을 반환 
3. 그러면 React가 함수가 반환한 스냅샷과 일치하도록 화면을 업데이트


## `state`와 렌더링

- state를 컴포넌트 외부에 저장 → 렌더링 중의 `state`는 "스냅샷"으로 고정된다.

컴포넌트가 렌더링되면 그 시점의 state 값을 가져와서 고정된 스냅샷으로 사용한다.
- `state`를 변경하면 **다음 렌더링에서만 업데이트된 값**을 사용한다.
- 렌더링 도중 이벤트 핸들러에서 사용하는 `state` 값은 항상 그 렌더링 당시의 값이다.

## 이벤트 핸들러에서 `state` 변화
``` jsx
        <button onClick={() => {
          setNumber(number + 1);
          setNumber(number + 1);
          setNumber(number + 1);
        }}>+3</button>
        
```
여기서 버튼을 클릭하면 setNumber를 세 번 호출했으니 number가 3 증가할 것 같지만.....

실제로는 1만 증가한다.

 setNumber(number + 1)가 호출된 후에도 number의 값은 여전히 0이며,
 
  세 번의 setNumber는 모두 동일한 값에서 시작한 것이다

> 같은 렌더링에서 여러 번 `setState`를 호출해도, **기존 렌더링의 `state` 값을 참조**하므로 즉시 반영되지 않는다.
        

##  비동기 동작과 `state`
### 1. 첫 번째 예시: 클릭 시 경고창에 표시되는 값
``` jsx

import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 5);
        alert(number);
      }}>+5</button>
    </>
  )
}
```
이 코드에서 버튼을 클릭하면 setNumber(number + 5)로 number를 5 증가시키고, 바로 alert(number)로 number를 경고창에 표시한다.

여기서 경고창에 표시되는 값은 ??

0 이다....

왜냐하면, setNumber(number + 5)는 React에 "number를 5로 업데이트해줘!"라고 요청할 뿐이지, 즉시 값을 변경하지 않는다.

React는 컴포넌트를 다시 렌더링하면서 업데이트된 값을 보여준다.

하지만, alert(number)는 렌더링 전의 state 값(이 경우 0)을 사용해서 버튼을 클릭한 시점에서의 스냅샷 값을 보여주는 것!!!


### 2. 두 번째 예시: 3초 뒤 경고창 표시
``` jsx

import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 5);
        setTimeout(() => {
          alert(number);
        }, 3000);
      }}>+5</button>
    </>
  )
}
```
여기서 버튼을 클릭하면:

setNumber(number + 5)로 number를 5로 업데이트 요청

3초 후 alert(number)로 number를 경고창에 표시한다.

경고창에 표시되는 값은..??


0이다

왜냐하면, setTimeout 안의 코드는 버튼 클릭 시점에 생성된 state 값을 "기억"한다.

React가 state를 업데이트하더라도, setTimeout은 **이미 고정된 스냅샷 값(0)**을 사용한다.

### 3. 왜 이런 일이 일어날까?

- React에서는 이벤트 핸들러나 비동기 코드가 렌더링 당시의 state 스냅샷을 사용하도록 설계되어 있다.

- 렌더링 시점에 state는 "고정"됩니다.

- setNumber는 state를 변경하는 "요청"만 할 뿐, 즉시 값을 바꾸지 않다.

따라서 비동기 코드나 이벤트 핸들러에서는 최신 state를 "기억"하지 않고, 그 렌더링 당시의 값을 사용하게 되는 것입니다~~

### 4. 최신 state를 사용하려면?
React에서 최신 state 값을 읽으려면, state 갱신 함수를 사용해야함!

이전 값을 받아와 갱신하면 최신 값을 안전하게 사용할 수 있다.

``` jsx
<button onClick={() => {
  setNumber((prevNumber) => prevNumber + 5);
  setTimeout(() => {
    alert(prevNumber + 5); // 최신 값 기반으로 경고창 표시
  }, 3000);
}}>+5</button>
```
위처럼 setNumber의 함수형 업데이트를 사용하면, 최신 state 값을 안전하게 사용할 수 있다.
