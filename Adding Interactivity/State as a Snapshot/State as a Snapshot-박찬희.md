# State as a Snapshot

State 변수는 스냅샷처럼 동작합니다. 변수를 설정하여도 이미 가지고있는 state변수는 변경되지 않고 리랜더링이 진행됩니다.

## state를 설정하면 랜더링이 동작합니다

인터페이스가 이벤트에 반응하려면 state를 업데이트 해야 합니다. 
```jsx
import { useState } from 'react';

export default function Form() {
  const [isSent, setIsSent] = useState(false);
  const [message, setMessage] = useState('Hi!');
  if (isSent) {
    return <h1>Your message is on its way!</h1>
  }
  return (
    <form onSubmit={(e) => {
      e.preventDefault();
      setIsSent(true);
      sendMessage(message);
    }}>
      <textarea
        placeholder="Message"
        value={message}
        onChange={e => setMessage(e.target.value)}
      />
      <button type="submit">Send</button>
    </form>
  );
}

function sendMessage(message) {
  // ...
}
```
 위 예시의 버튼을 클릭하면 발생하는 일
 1. `onSubmit` 이벤트 핸들러 실행
 2. `setIsSent(true)`가 `isSent`를 `true`로 설정, 새로운 랜더링에 큐를 넣음
 3. React는 새로운 `isSent`값에 따라 컴포넌트를 다시 랜더링

 ## 랜더링은 그 시점의 스냅샷을 찍습니다
 랜더링 진행시 해당 함수에서 반환하는 JSX는 시간상 UI의 스냅샷과 같습니다. prop, 이벤트 핸들러, 로컬 변수는 모두 랜더링 당시의 state를 사용해 계산됩니다. 

 사진이나 동영상 프레임과 달리 UI스냅샷은 대화형입니다. 이것은 이벤트 핸들러와 같은 로직이 포함된다는 뜻입니다.  그러면 React는 스냅샷과 일치하도록 화면을 업데이트하고 이벤트 핸들러를 연결. 버튼을 누르면 JSX의 클릭 핸들러가 진행됩니다.

 React의 컴포넌트 리랜더링
 1. React가 함수를 다시 호출
 2. 함수가 새로운 JSX 스냅샷 반환
 3. React가 함수가 반환한 스냅샷과 일치하도록 화면 업데이트


 컴포넌트 메모리로써 state는 함수가 반환된 후 사라지는 일반 변수와는 다르게 함수 외부 React 자체에 존재합니다. 따라서 React가 state를 업데이트하게 되면 스냅샷을 찍어 컴포넌트는 해당 랜더링의 state 값을 사용해 계산된 새로운 props 세트와 이벤트 핸들러가 포함된 UI의 스냅샷을 JSX에 반환합니다.

 이는 같은 `setter` 함수를 한 번에 반복해서 호출하더라도, 스냅샷은 하나였기 때문에 반복 시행된 만큼 state가 업데이트 되지 않고 한 번만 업데이트 된다는 것입니다.

## 시간 경과에 따른 State

 또한 타이머를 설정하여 랜더링 된 이후 함수가 실행되도록 하더라도 이전 스냅샷에 대해 실행되는 함수이기 때문에 리랜더링 된 컴포넌트는 신경쓰지 않습니다.

 