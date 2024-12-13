# Effect 의존성 제거하기

Effect의 의존성을 제거하기 위해서는 의존성이 아니라는 것을 증명해야합니다. Effect의 의존성은 선택할 수 없기 때문입니다.
Effect는 의존성 목록의 반응형 값에 의해 실행되고 불필요한 의존성 목록들이 선언되면 자주 실행되고 혹은 무한 루프를 생성할 수 있습니다.

## 의존성을 변경하려면 코드를 변경해야합니다.

Effect의 의존성을 변경하려면 흐름을 파악하고 코드를 변경해야합니다.

1. 먼저 Effect의 코드 또는 반응형 값 선언 방식을 변경
2. 변경 코드에 맞게 의존성 조정
3. 의존성 목록이 마음에 들지 않으면 1번으로 돌아가기(이후 다시 코드 변경)
의존성을 변경하기 위해서는 우선 주변 코드를 변경해야합니다. 의존성 목록은 Effect의 코드에서 사용하는 모든 반응형 값의 목록이라고 생각하면 쉽습니다. 

## 불필요한 의존성 제거하기

- 이벤트 핸들러로 옮기기
  Effect에서는 반응형 값이 변할 때 마다 실행되어야하는 반응형 로직들이 들어가 있습니다.
  하지만 이벤트 핸들러의 경우에는 반응형 값이 바뀜에 따라 매번 실행되지 않아도 되는, 특정한 행동을 해야 실행을 하는 로직이기 때문에, 우선 Effect에 이벤트 핸들러로 옮길 수 있는 로직이 있는지 확인해야합니다.

- Effect와 관련 없는 작업 수정하기
  Effect에서 이벤트 핸들러를 제거했다면, 이제 서로 분리할 수 있는 Effect를 찾아봅시다.
  Effect에서 서로 관련이 없는 작업을 하고 있는지 확인해볼 필요가 있습니다.
  예를 들어, 채팅 방에 접속하면 동기화하는 Effect가 있는데 그 안에 시계를 표현하기 위해 매 초 실행되어야하는 Effect가 있다고 생각해봅시다.
  이렇게 되면 Effect는 매 초마다 채팅방을 동기화를 시도할 것입니다. 이것은 불필요한 행동입니다!

- Effect 내부에서 state 읽지 않기
이 Effect는 새 메시지가 도착할 때마다 새로 생성된 배열로 `messages` state 변수를 업데이트합니다.
```jsx
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      setMessages([...messages, receivedMessage]);
    });
    return () => connection.disconnect();
  }, [roomId, messages]); // ✅ All dependencies declared
  // ...
    useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      setMessages(msgs => [...msgs, receivedMessage]);
    });
    return () => connection.disconnect();
  }, [roomId]);
  // state를 제외하고 업데이트 함수로 변경경
}
```
해당 Effect는 메시지를 수신할 때마다 수신된 메시지를 포함하는 새로운 `messages` 배열로 리랜더링됩니다.
또한 `message`가 Effect의 의존성 배열에 속해있기 때문에 Effect도 다시 동기화됩니다.
즉 새로운 메시지가 올 때마다 채팅이 다시 연결되는 것입니다.
이 문제를 해결하기 위해서는 Effect에서 state를 읽으면 안됩니다. 예제에서 볼 수 있듯 새로운 업데이트 함수를 만들어 setter 함수를 통해 state로 넣어주기만 하면 됩니다. 이제는 새로운 메시지가 올때마다 채팅방에 다시 연결하지 않을 겁니다.

### 값의 변경에 반응하지 않고 값을 읽고 싶을 떄
> 아직 React에서 실험중인 API로 안정적이지 않습니다.

이전 Effect에서 이벤트 분리할 때 알아봤던 Effect 이벤트입니다. 
이것은 반응형 값을 Effect 내부에서 다루고 싶지만 해당 반응형 값을 의존성 목록으로 사용하고 싶지 않을 때 사용합니다.

다음은 컴포넌트가 이벤트 핸들러를 props로 받을 때 발생할 수 있는 문제입니다.
```jsx
// ChatRoom 컴포넌트는 부모에게 랜더링마다 매번 다른 onReceiveMessage를 받습니다.
<ChatRoom
  roomId={roomId}
  onReceiveMessage={receivedMessage => {
    // ...
  }}
/>

function ChatRoom({ roomId, onReceiveMessage }) {
  const [messages, setMessages] = useState([]);

  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      onReceiveMessage(receivedMessage);
    });
    return () => connection.disconnect();
  }, [roomId, onReceiveMessage]); // ✅ All dependencies declared
  // ...
  ```
  매번 `onReceiveMessage` 함수가 바뀌기 때문에 의존성이므로 부모가 재랜더링할 때마다 Effect는 다시 동기화됩니다. 이 문제를 해결하기 위해 Effect이벤트로 호출을 감싸보겠습니다.

  ```jsx
  function ChatRoom({ roomId, onReceiveMessage }) {
  const [messages, setMessages] = useState([]);

  const onMessage = useEffectEvent(receivedMessage => {
    onReceiveMessage(receivedMessage);
  });

  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      onMessage(receivedMessage);
    });
    return () => connection.disconnect();
  }, [roomId]); // ✅ All dependencies declared
  ```
  다음과 같이 `onReceiveMessage`를 state처럼 Effect 이벤트로 감싸서 사용했습니다. 이제 더이상 의존성으로 지정할 필요가 없으며 부모 컴포넌트가 리랜더링될 때 다른 함수를 전달하더라도 더이상 채팅이 다시 연결되지 않습니다.

### 일부 반응형 값이 의도치 않게 변경될 때

  Effect가 특정 값에 반응하기를 원하지만 그 값이 원하는 것보다 더 자주 변경되어 사용자의 관점에서 실제 변경사항을 반영하지 못할 수도 있습니다.
  다음 예시는 컴포넌트 본문에 `options` 객체를 생성 후 Effect 내부에서 객체를 읽고 있습니다.

```jsx
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  // 문제를 보여주기 위해 일시적으로 린터를 비활성화합니다.
  // eslint-disable-next-line react-hooks/exhaustive-deps
  const options = {
    serverUrl: serverUrl,
    roomId: roomId
  };

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [options]);

  return (
    <>
      <h1>Welcome to the {roomId} room!</h1>
      <input value={message} onChange={e => setMessage(e.target.value)} />
    </>
  );
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  return (
    <>
      <label>
        Choose the chat room:{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">general</option>
          <option value="travel">travel</option>
          <option value="music">music</option>
        </select>
      </label>
      <hr />
      <ChatRoom roomId={roomId} />
    </>
  );
}
```
이 예제는 `roomId`가 변경되면 Effect가 새 `options`로 채팅에 다시 연결됩니다. 이제 보기에는 문제가 없어보이지만 코드에 문제가 있습니다.
바로 input 에 메시지를 입력하게 되면 채팅방 동기화가 다시 진행됩니다.
메시지를 입력하는 행동은 `message` state 변수만 업데이트합니다. 이는 채팅 연결에 영향을 미치지 않아야합니다. 하지만 `message`가 업데이트 될 때마다 컴포넌트가 리랜더링되어 코드가 처음부터 다시 실행됩니다.
`options`는 객체입니다. 컴포넌트는 리랜더링 될 때마다 새로운 `options`객체를 생성하게 되고, 이 객체는 랜더링 중에 생성되었기 때문에 기존의 `options`객체와는 다른 객체로 인식됩니다. 따라서 Effect는 다시 동기화하고 사용자가 입력할 때 채팅이 다시 연결됩니다.
따라서 객체 및 함수 의존성으로 인해 Effect가 필요 이상으로 자주 재동기화될 수 있다는 점을 기억해야합니다.

- 정적 객체와 함수를 컴포넌트 외부로 이동
  객체가 props 및 state에 의존하지 않는 경우에는 해당 객체를 컴포넌트 외부로 이동할 수 있습니다.
```jsx
  const options = {
  serverUrl: 'https://localhost:1234',
  roomId: 'music'
};
function ChatRoom() {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, []);
```
  이제는 리랜더링되어도 동기화하지 않습니다. 하지만 의존성이 없기 때문에 다른 방으로 동기화도 할 수 없을 것입니다.

- Effect 내에서 동적 객체 및 함수 이동
객체가 `roomId` props 처럼 반응형 값에 의존한 경우, Effect 코드 내부로 이동 시킬 수 있습니다.
```jsx
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);
```
이제 `options`가 Effect 내부에서 선언되었으므로 `options`가 Effect의 의존성이 되지 않습니다. 대신 `roomId`가 유일한 반응형 값이 될것입니다. 이제 객체를 의존성으로 가지고 있지 않기 때문에 리랜더링 되더라도 Effect는 다시 실행되지 않을 것입니다.

- 객체에서 원시 값 읽기
  props에서 객체를 받을 수도 있습니다.
  그 객체를 Effect에서 의존성으로 사용한다면 또 다시 아까와 같은 문제에 직면하게 될것입니다.
  이때는 객체에서 정보를 읽어 객체 및 함수 의존성을 피해야합니다.
  ```jsx
  function ChatRoom({ options }) {
  const [message, setMessage] = useState('');

  const { roomId, serverUrl } = options;
  useEffect(() => {
    const connection = createConnection({
      roomId: roomId,
      serverUrl: serverUrl
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]);
  ```
  약간의 반복적이라 번거로울 수 있습니다. 하지만 Effect가 실제로 어떤 정보에 의존하는지 명확하게 알 수 있습니다. 부모 컴포넌트에 의해 의도치 않게 객체가 다시 생성된 경우 채팅이 다시 연결되지 않습니다만 새로 만든 `options` 객체의 `options.roomId`, `options.serverUrl`이 실제로 다른 경우 채팅이 다시 연결됩니다.

- 함수
  함수에 대해서도 각각 동일하게 접근할 수 있습니다.
  앞서 다뤘던 정적 객체, 동적 객체를 다룬 것처럼 함수 또한 외부에 선언하고, 내부에 선언하면 동일한 결과를 얻을 수 있습니다. 
  또한 마지막으로 다뤘던 객체에서 원시 값을 읽은 것과 같이 함수에서 원시 값을 읽어내면 같은 결과를 얻어낼 수 있습니다.
  단, 순수 함수여야 한다는 점을 기억해야합니다.