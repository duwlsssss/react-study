### 반응형 effects의 생명주기

마운트, 업데이트 또는 언마운트 되는 컴포넌트와 달리
effect는 동기화를 시작하고 나중에 동기화를 중지하는 두 가지 작업만 할 수 있음. 즉 컴포넌트와는 별도의 생명주기를 가짐 

React는 effect의 의존성을 올바르게 지정했는지 확인하는 린터 규칙을 제공하여 effect가 최신 props와 state에 동기화됨.

- 각 effect는 별도의 독립적인 동기화 프로세스를 나타냄
한 effect를 삭제해도 다른 effect의 로직이 깨지지 않음
```js
function ChatRoom({ roomId }) {
  //roomId 변화에 따라
  //방문을 기록
  useEffect(() => {
    logVisit(roomId);
  }, [roomId]);

  //방문을 연결
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    // ...
  }, [roomId]);
  // ...
}
```

React는 모든 반응형 값이 effect에 종속성으로 선언됐는지 확인함   

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

function ChatRoom({ roomId }) { // roomId는 반응형
  const [serverUrl, setServerUrl] = useState('https://localhost:1234'); // serverUrl는 반응형

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, []); // 🔴 Lint Error 
  }, [serverUrl, roomId]); // 🟢 모든 반응형 값이 종속성으로 선언됨
  //...
```

```js
function ChatRoom() {
  useEffect(() => {
    const serverUrl = 'https://localhost:1234'; // serverUrl는 반응형이 아님 - 렌더링 중에 계산되지 않음
    const roomId = 'general'; // roomId는 반응형이 아님
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, []); // ✅ 모든 반응형 값이 종속성으로 선언됨
  // ...
}
```