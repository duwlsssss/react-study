### Effect에서 이벤트 분리하기 

Effect의 코드 일부가 반응형이 아니길 원할 수도 있다. 
그땐 Effect Event에 반응하지 않는 로직을 추출하는 React 훅 `useEffectEvent`를 사용할 수 있다.
이렇게 생성된 함수는 최신 상태를 보장하고, 컴포넌트가 다시 렌더링되어도 재생성되지 않음
⚠ useEffectEvent는 아직 개발중인 API임 / Effect 내부에서만 호출해야함 / 다른 컴포넌트나 Hook에 전달 불가

```js
const onSomething = useEffectEvent(callback)
```

```js
import { useState, useEffect } from 'react';
import { createConnection, sendMessage } from './chat.js';
import { showNotification } from './notifications.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId, theme }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      showNotification('연결됨!', theme);
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, theme]); // theme이 변하면 불필요하게 연결이 재설정된다. -> showNotification('연결됨!', theme);부붐이 반응형이 아니었음 좋겠다면?

  return <h1>{roomId} 방에 오신 것을 환영합니다!</h1>
}

  // useEffectEvent훅 사용
  // 반응형 변수 theme이 바뀌어도 Effect가 재실행되지 않으나 바뀌는 theme을 트래킹할 수 있다.
  const onConnected = useEffectEvent((connectedTheme) => { //매개변수를 넘겨 Effect에서 호출됐을 당시의 theme을 사용 가능 / 안넘기면 가장 최근의 theme 사용
    showNotification('연결됨!', connectedTheme);
  });

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      onConnected(connectedTheme);
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);

```

