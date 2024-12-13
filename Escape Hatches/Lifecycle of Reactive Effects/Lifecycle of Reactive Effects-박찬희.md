# 반응형 effects의 생명주기

effect는 컴포넌트와 다른 생명주기를 가집니다. effect는 동기화 시작과 동기화 중지 단 두가지 작업만 할 수 있습니다.

## effect의 생명주기
모든 React 컴포넌트는 동일한 생명주기를 거칩니다.
- 컴포넌트는 화면에 추가될 때 마운트됩니다.
- 컴포넌트는 일반적으로 상호작용에 대한 응답으로 새로운 props나 state를 수신하면 업데이트됩니다.
- 컴포넌트가 화면에서 제거되면 마운트가 해제됩니다.
하지만 effect의 생명주기를 컴포넌트의 생명주기와 동일시하는 것은 좋지 않습니다. 
effect는 외부 시스템을 현재 props 및 state와 동기화하는 방법을 설명합니다.

다음 예는 컴포넌트를 채팅 서버에 연결하는 effect입니다.

```jsx
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  useEffect(() => {
    // 동기화 시작
    const connection = createConnection(serverUrl, roomId);
    connection.connect(); 
    return () => {
      connection.disconnect(); // 동기화 중지
    };
  }, [roomId]);
  // ...
}
```
직관적으로 React는 컴포넌트가 마운트 될 때 동기화를 시작하고 마운트 해제될 때 중지할것이라 생각할 수 있지만, 컴포넌트가 마운트 된 상태에서도 동기화를 여러 번 시작하고 중지해야 할 수도 있습니다.

## 동기화가 두 번 이상 수행되어야 하는 경우
```jsx
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId /* "general" */ }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // "general" 방에 연결(동기화)
    connection.connect();
    return () => {
      connection.disconnect(); // "general" 방에서 연결 해제(동기화 중지)
    };
  }, [roomId]);// roomId가 변경될 때 마다 반복
  return <h1>Welcome to the {roomId} room!</h1>;
}
```
해당 ChatRoom 컴포넌트는 사용자가 드롭다운에서 선택한 `roomId` prop을 받습니다.
사용자가 `general`대화방을 선택했을 때, React가 effect를 실행하여 동기화를 시작합니다.
그 뒤 드롭다운으로 다른 방을 선택했다고 가정해봅시다. 그렇다면 먼저 React가 UI를 업데이트합니다.
이때 `general`대화방에 연결되어 있는 상태에서 다른 방에 연결하고자 합니다. 이때 effect는 `roomId` prop이 변경되었기 때문에 effect가 수행한 `general` 방에 동기화가 더이상 UI가 일치하지 않습니다.

이때 React는 두 가지 작업을 수행합니다.
1. 이전 roomId와의 동기화를 중지.
2. 새 roomId와 동기화 시작
이것은 effect의 본문에 이미 명시되어 있습니다. 

## React가 effect를 재동기화하는 방법
전과 이어 `ChatRoom` 컴포넌트의 `roomId` prop은 새로운 값을 받았습니다. 다시 선택한 방이 `travel`이라하면 이전 `general`에서 `travel`로 방을 다시 동기화해야합니다.

동기화 중지를 위해 React는 `general` 방에 연결 후 effect가 반환한 cleanup함수를 호출합니다.
cleanup 함수의 호출에 따라 `general`방과의 연결은 끊기게되고 이젠 `travel` 채팅방과 동기화를 시작합니다.
이 `travel` 채팅방 또한 다음 `roomId`가 선택되고 cleanup 함수가 호출되기 전까지 동기화될 것입니다.
이 모든 과정이 전에 다뤘던 예제에서 확인 할 수 있습니다.

마지막으로 사용자가 다른 화면으로 이동하게 되면 `ChatRoom`이 마운트를 해제합니다. 더 이상 연결 상태를 유지할 필요가 없어지면 React는 effect 동기화를 중지하고 마지막으로 연결되어있던 `roomId`에 해당되는 채팅방과의 연결을 끊게 될겁니다.

## effect 관점에서 생각하기

`ChatRoom` 컴포넌트 과정에서는 다음과 같은 일들이 일어났습니다.
1. `roomId`가 `general`로 설정되어 마운트 된 `ChatRoom`
2. `roomId`가 `travel`로 설정되어 업데이트 된 `ChatRoom`
3. 마운트 해제 된 `ChatRoom`

위와 같은 컴포넌트의 생명주기에 effect는 다음과 같은 일을 했습니다.
1. effect가 `general` 대화방에 연결
2. `general` 방에서 연결이 끊어지고, `travel` 방에 연결된 effect
3. `travel` 방에서 연결이 끊어진 effect

앞서 다뤘던 예제를 다시 확인해봅시다.
```jsx
  useEffect(() => {
    // roomId로 지정된 방에 연결된 effect...
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      // ...연결이 끊어질 때까지
      connection.disconnect();
    };
  }, [roomId]);
  ```
  이 코드의 구조는 어떤 일이 일어났는지 겹치지 않는 시간의 연속으로 보는 데 영감을 줄 수 있습니다.

  1. `general` 방에 연결된 effect(연결이 끊어질 떄까지)
  2. `travel` 방에 연결된 effect(연결이 끊어질 때까지)

컴포넌트의 관점에서 생각하는 것처럼 특정 시점에 실행되는 "콜백" 또는 "생명주기 이벤트"로 생각하기 쉽습니다. 하지만 이런 사고방식은 빠르게 복잡해지므로 피하는 것이 좋습니다.

대신 항상 한 번의 시작/중지 사이클에 집중하는 것이 좋습니다.
동기화를 시작하는 방법과 중지하는 방법만 설명하면 됩니다.
이 작업을 잘 수행하면 필요한 횟수만큼 effect를 싲가하고 중지할 수 있는 탄력성을 확보할 수 있습니다.

## React가 effect를 다시 동기화될 수 있는지 확인하는 방법
다음은 직접 확인해볼 수 있는 예시입니다.
```jsx
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);
  return <h1>Welcome to the {roomId} room!</h1>;
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  const [show, setShow] = useState(false);
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
      <button onClick={() => setShow(!show)}>
        {show ? 'Close chat' : 'Open chat'}
      </button>
      {show && <hr />}
      {show && <ChatRoom roomId={roomId} />}
    </>
  );
}
export function createConnection(serverUrl, roomId) {
  // 실제 구현은 실제로 서버에 연결됩니다.
  return {
    connect() {
      console.log('✅ Connecting to "' + roomId + '" room at ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ Disconnected from "' + roomId + '" room at ' + serverUrl);
    }
  };
}

```
컴포넌트가 처음 마운트 될 때 3개의 로그가 표시됩니다.
1. ✅ https://localhost:1234... 에서 "general" 방에 연결 중입니다. (개발 전용)
2. ❌ https://localhost:1234에서 "일반" 방에서 연결 해제되었습니다. (개발 전용)
3. ✅ https://localhost:1234... 에서 "general" 방에 연결 중입니다.

첫 두 로그는 개발 전용입니다. 
React는 개발 단계에서 즉시 강제로 동기화를 수행하여 effect가 다시 동기화 할 수 있는지 확인합니다.
마치 도어록 작동을 확인하기 위해 문을 열었다 다시 닫는 것을 생각해볼 수 있습니다.
React는 개발 과정에서 effect를 한 번 더 시작하고 중지하여 정리를 잘 구현했는지 확인합니다..

## React가 effect를 다시 동기화해야 하는 것을 인식하는 방법
```jsx
function ChatRoom({ roomId }) { // roomId prop은 시간이 지남에 따라 변경될 수 있습니다.
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // 이 effect는 roomId를 읽습니다.
    connection.connect();
    return () => {
      connection.disconnect();
    };
    
  }, [roomId]); // 따라서 React에 이 effect가 roomId에 "의존"한다고 알려줍니다.
  // ...
  ```
  1. `roomId`가 `prop`이므로 시간이 지남에 따라 변경될 수 있다는 것을 알고 있습니다.
  2. effect가 `roomId`를 읽는걸 알았습니다. 이로 인해 변경될 수 있는 값에 따라 로직이 달라집니다.
  3. 그렇기 때문에 `roomId`를 변경되면 다시 동기화 되도록 종속성으로 지정했습니다.

  ## 각 effect는 별도의 동기화 프로세스를 나타냅니다.
```jsx
function ChatRoom({ roomId }) {
  useEffect(() => {
    logVisit(roomId);// 관련 없는 로직!
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId]);
  // ...
}
```
이 로직은 이미 작성한 effect와 동시에 실행되어야하므로 관련 없는 로직을 추가하면 안됩니다.
사용자가 회의실을 방문할 때 분석 이벤트를 전송하고 싶을 수 있습니다. 위 예제에서 확인 할 수 있습니다.
위 예제대로 된다면 effect가 다시 동기화되면 의도하지 않은 동일한 방에 대해 `logVisit(roomId)`도 호출합니다. 방문 기록하는 것과 연결은 별개의 프로세스입니다. 두 개의 개별 effect로 작성해야합니다.
```jsx
function ChatRoom({ roomId }) {
  useEffect(() => {
    logVisit(roomId);
  }, [roomId]);

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    // ...
  }, [roomId]);
  // ...
}
```
코드의 각 effect는 별도의 독립적인 동기화 프로세스를 나타내야합니다.

위의 예시에서는 한 effect를 삭제해도 다른 effect의 로직이 깨지지 않습니다. 이는 서로 다른 것을 동기화하므로 분리하는 것이 합리적이라는 것을 나타냅니다. 반면에 일관된 로직을 별도의 effect로 분리하면 코드가 “더 깔끔해” 보일 수 있지만 유지 보수가 더 어려워집니다. 따라서 코드가 더 깔끔해 보이는지 여부가 아니라 프로세스가 동일하거나 분리되어 있는지를 고려해야 합니다.

## 반응형 값에 반응하는 effect

effect에서 두 개의 변수 (`serverUrl` 및 `roomId`)를 읽지만 종속성으로 `roomId`만 지정했습니다.
```jsx
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId]);
  // ...
}
```
여기서 `serverUrl`은 왜 종속성이 될 필요가 없을까요?

그 이유는 재 랜더링으로 인해 `serverUrl`이 변경되지 않기 때문입니다. 컴포넌트가 몇 번이나 다시 랜더링하든 동일합니다.

반면 `roomId`는 다시 랜더링할 때 달라질 수 있습니다. 컴포넌트 내부에서 선언된 Props, state 및 기타값은 랜더링 중에 계산되고 React 데이터 흐름에 참여하기 때문에 반응형입니다.

```jsx
function ChatRoom({ roomId }) { // Props는 시간이 지남에 따라 변화합니다.
  const [serverUrl, setServerUrl] = useState('https://localhost:1234'); // State는 시간이 지남에 따라 변화합니다.

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // effect는 Props와 state를 읽습니다.
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId, serverUrl]); // 따라서 이 effect는 props와 state에 "의존"한다고 React에 알려줍니다.
  // ...
}
```
`serverUrl`을 종속성으로 포함하면 effect가 변경된 후 다시 동기화 되도록할 수 있습니다.

## 빈 종속성이 있는 effect의 의미
`serverUrl`과 `roomId` 모두 컴포넌트 외부로 이동하면 effect 코드는 어떠한 반응형 값도 사용하지 않아 종속성이 비어 있을 수 있습니다.
```jsx
const serverUrl = 'https://localhost:1234';
const roomId = 'general';

function ChatRoom() {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, []); // ✅ 선언된 모든 종속성
  // ...
}
```

이 경우 컴포넌트가 마운트 될 때만 채팅방에 연결되고 컴포넌트가 마운트 해제될 때만 연결이 끊어지는 것을 의미합니다. 물론 개발 단계에서는 한 번 더 동기화 합니다.

## 컴포넌트 본문에서 선언된 모든 변수는 반응형입니다.

props와 state만 반응형 값은 아닙니다. 이것을 통해 계산하는 값도 반응형입니다.
props와 state가 변경되면 컴포넌트가 다시 랜더링되고 그로부터 계산된 값도 변경됩니다.
이 때문에 effect에서 사용하는 컴포넌트 본문의 모든 변수는 effect 종속성 목록에 있어야합니다.

```jsx
function ChatRoom({ roomId, selectedServerUrl }) { // roomId는 반응형입니다.
  const settings = useContext(SettingsContext); // settings는 반응형입니다.
  const serverUrl = selectedServerUrl ?? settings.defaultServerUrl; // serverUrl는 반응형입니다.
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // effect는 roomId 와 serverUrl를 읽습니다.
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId, serverUrl]); // 따라서 둘 중 하나가 변경되면 다시 동기화해야 합니다!
  // ...
}
```
이 예시에서 `serverUrl`은 prop이나 state 변수가 아닙니다. 랜더링 중에 계산하는 일반 변수입니다. 하지만 랜더링 중에 계산되므로 재 랜더링으로 인해 변경 될 수 있습니다. 따라서 바로 반응형인것입니다.

컴포넌트 내부의 모든 값은 반응형입니다. 모든 반응형 값은 다시 랜더링할 때 변경될 수 있으므로 반응형 값을 effect의 종속 요소로 포함해야합니다.

## React는 모든 반응형 값을 종속성으로 지정했는지 확인합니다.
린터가 React에 대해 구성된 경우, effect의 코드에서 사용되는 모든 반응형 값이 종속성으로 선언되었는지 확인합니다.
```jsx
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

function ChatRoom({ roomId }) { // roomId는 반응형입니다.
  const [serverUrl, setServerUrl] = useState('https://localhost:1234'); // serverUrl는 반응형입니다.

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, []); // <-- 여기 무언가 잘못되었습니다!
  // }, [serverUrl, roomId]);  ✅ 선언된 모든 종속성으로 변경 시 린트 사라짐
}
```
여기서 종석성에 대해서 React 오류가 발생한 것 처럼 보일 수 있지만 실제로는 코드의 버그를 지적하는 것입니다. 해당 예시의 경우 `roomId`와 `serverUrl`은 반응형이라 변경 가능하지만 변경 시 effect를 다시 동기화하는 것은 설정되어있지 않습니다. 따라서 두 반응형 값을 종속성으로 선언해야하기 때문에 린트에서 오류를 발생시키는 것입니다.

>어떤 경우에는 컴포넌트 내부에서 값이 선언되더라도 절대 변하지 않는다는 것을 React가 알고 있습니다. 예를 들어, `useState`에서 반환되는 `set` 함수와 `useRef`에서 반환되는 ref 객체는 안정적이며, 다시 렌더링해도 변경되지 않도록 보장됩니다. 안정된 값은 반응하지 않으므로 목록에서 생략할 수 있습니다. 이러한 값은 변경되지 않으므로 포함해도 상관없습니다.

## 다시 동기화하지 않으려는 경우 어떻게 해야 하나요?

위 예제에서 `roomId`와 `serverUrl`이 종속성으로 나열해 해결했지만 사실 이 반응형 값들이 반응형 값이 아닌 경우 컴포넌트 외부로 옮겨 선언한다면 린터에게 재랜더링의 결과로 변경될 수 없다는 것을 증명할 수 있습니다. 즉 종속성이 될 필요가 없어지는 것입니다.

```jsx
const serverUrl = 'https://localhost:1234'; // serverUrl는 반응형이 아닙니다.
const roomId = 'general'; // roomId는 반응형이 아닙니다.

function ChatRoom() {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, []); // ✅ 선언된 모든 종속성
  // ...
}

function ChatRoom() {
  useEffect(() => {
    const serverUrl = 'https://localhost:1234'; // serverUrl는 반응형이 아닙니다.
    const roomId = 'general'; // roomId는 반응형이 아닙니다.
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, []); // ✅ 선언된 모든 종속성
  // ...
} // effect 내부 선언!
```
또는 내부에서 특정 값으로 선언한다면 랜더링 중 계산되지 않으므로 effect 내부에서 선언할 수 있습니다.

effect는 반응형 코드 블록입니다. 상호작용 당 한 번만 실행되는 이벤트 핸들러와 달리 effect는 동기화가 필요할 때마다 실행됩니다.

종속성을 선택할 수 없습니다. 종속성은 effect에서 읽은 모든 반응형 값이 포함되어야합니다. 린터가 이를 강제할 것입니다. 하지만 이것 때문에 무한 루프같은 문제가 발생하거나 너무 자주 effect가 동기화 될 수 있습니다. 이때 린터를 억제하여 해결하는 것은 옳지 않습니다. 다음과 같은 방법을 시도해봅시다.

- effect가 독립적인 동기화 프로세스를 나타내는지 확인해보기. 
  effect가 아무것도 동기화하지 않는다면 불필요한 effect일 수 있습니다. 또한 여러 개의 독립적인 것을 동기화하는 경우 분할해야합니다.

- **props나 state에 반응하지 않고 effect를 다시 동기화하지 않고 최신 값을 읽으려면** effect를 반응하는 부분(effect 유지)과 반응하지 않은 부분(effect 이벤트로 추출 가능한것)을 분리하면 됩니다.

- 객체와 함수를 종속성으로 사용하지 마세요. 랜더링 중 오브젝트와 함수를 생성 후 effect에서 읽을 때 랜더링마다 달라집니다. 이는 매번 effect를 다시 동기화해야합니다.