# Effect에서 이벤트 분리하기

이벤트 핸들러는 같은 상호작용을 반복해야 재실행하고 Effect는 prop이나 state 변수 등 읽은 값이 마지막 랜더링과 다르면 다시 동기화합니다.

## 이벤트 핸들러와 Effect 중에 선택하기

이전 시간에 다뤘던 채팅방 컴포넌트를 예를 들어보겠습니다. 채팅방을 구현할 때 다음것을 고려해봅시다.
1. 채팅방 컴포넌트는 선택된 채팅방에 자동으로 연결해야합니다.
2. "전송"버튼을 클릭하면 채팅에 메시지를 전송합니다.

### 이벤트 핸들러는 특정 상호작용에 응답으로 실행된다.

사용자 관점에서 메시지는 "전송" 버튼을 클릭했기 때문에 전송되어야합니다. 다른 이유로 메시지가 전송되면 그것은 의도된것이 아닙니다. 따라서 메시지 전송은 이벤트 핸들러가 되어야합니다.
이벤트 핸들러는 특정 상호작용을 처리해야한다는 것을 기억하세요.

### Effect는 동기화가 필요할 때마다 실행된다.

채팅방에 연결하는 것을 생각해봅시다. 이것은 특정한 상호작용에 의해 실행되는 것이 아니라 채팅방 화면으로 이동했기 때문에 화면을 보고 선택된 채팅 서버에 연결되어있어야합니다. 
채팅방 컴포넌트가 앱의 첫 화면이고 사용자가 아무런 상호작용을 하지 않은 경우라도 여전히 연결되어있어야합니다.
그렇기 떄문에 Effect를 사용해야합니다. Effect를 사용하면 특정 상호작용과는 상관없이 현재 선택된 채팅 서버와 항상 연결된 상태임을 확신할 수 있습니다.
Effect를 사용하기 때문에 사용자가 무엇을 하든 화면에 컴포넌트가 현재 선택된 방과 동기화된 상태를 유지하고 필요할 때마다 다시 연결할 것입니다.

```jsx
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);
} // 채팅방 연결
  function handleSendClick() {
    sendMessage(message);
  } // 메시지 전송
```

## 반응형 값과 반응형 로직

이벤트 핸들러는 버튼 클릭과 같이 항상 수동으로 트리거 되지만, Effect는 동기화 유지를 하기 위해 자주 실행 및 재실행 되기 때문에 자동으로 트리거됩니다.
컴포넌트 내부 props,state와 같은 변수를 반응형 값이라고 하고 반응형 값은 데이터 랜더링 과정에 관여합니다.
반응형 값이 변함에 따라 리랜더링 되고, 이에 따라 이벤트 핸들러와 Effect는 서로 다르게 반응합니다.
- 이벤트 핸들러 내부의 로직은 반응형이 아닙니다. 변화에 반응하지 않으면서 반응형 값을 읽을 수 있습니다. 즉 사용자가 상호작용을 반복하지 않는 이상 재실행되지 않습니다. 
메시지를 보내는 것을 생각해봅시다. 메시지 데이터가 바뀐다고 해서 메시지를 전송하고 싶은 것은 아닙니다. 
그렇기 때문에 메시지를 전송하는 로직은 반응형이어서는 안되며, 이벤트 핸들러에 속해야하는 것입니다.

- Effect 내부의 로직은 반응형입니다. 따라서 Effect는 반응형 값을 읽는 경우 그 값을 의존성으로 지정해야합니다. 이로 인해 값이 바뀔 때 React가 새로운 값으로 Effect 로직을 다시 실행합니다.
아까 다뤘던 채팅방과 동기화를 생각해봅시다. 사용자가 `roomId`를 바꾸는 것은 다른 방에 연결하고 싶다는 의미입니다. 그에 따라 Effect는 선택한 `roomId`에 해당하는 방으로 연결하는 반응형 로직을 가져야합니다.
따라서 반응형 값을 따라 결과가 바뀌어 다시 실행되어야 하기 때문에 Effect를 사용해야하고, Effect는 반응형 로직을 가져야한다는 것입니다.

## Effect에서 비반응형 로직 추출하기
반응형 로직과 비반응형 로직을 섞으려 한다면 까다로워집니다.
아래 예제는 사용자가 채팅에 연결할 때 알림을 보여주는 상황입니다. 올바른 색상의 알림을 보여주기 위해 props로부터 현재 테마(dark 또는 light)를 읽습니다.
```jsx
function ChatRoom({ roomId, theme }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      showNotification('연결됨!', theme);
    });
    connection.connect();
    return () => {
      connection.disconnect()
    };
  }, [roomId, theme]); // ✅ 모든 의존성 선언됨
  // ...
```
`theme`는 반응형 값이고 Effect가 읽는 모든 반응형 값은 의존성으로 선언되어야 하기 때문에 `theme`또한 Effect의 의존성으로 지정해야합니다.
하지만 `roomId`가 변경될 때만 채팅이 다시 연결되길 바라는 것과는 달리 `theme`도 의존성이기 때문에 dark테마와 light 테마를 전환할 때마다 채팅방이 다시 연결됩니다.
이로 인해 반응형 값을 가지고 Effect 내부에 있지만 반응형이 아니길 바랄때 다음과 같은 방법을 사용해봅시다.

### Effect 이벤트 선언하기
> 아직 안정적이지 않은 실험적인 API로 주의해야합니다.

`useEffectEvent`는 비반응형 로직을 Effect에서 추출하기 위한 특수한 Hook입니다.
```jsx
  const onConnected = useEffectEvent(() => {
    showNotification('연결됨!', theme);
  });
```
지금 `onConnected`를 EffectEvent로 선언했습니다. Effect 로직의 일부이지만 이벤트 핸들러와 비슷하게 동작합니다. 내부 로직은 반응형이 아니며 항상 props와 state의 최근값을 바라봅니다.

```jsx
import { useState, useEffect } from 'react';
import { experimental_useEffectEvent as useEffectEvent } from 'react';
import { createConnection, sendMessage } from './chat.js';
import { showNotification } from './notifications.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId, theme }) {
  const onConnected = useEffectEvent(() => {
    showNotification('연결됨!', theme);
  });

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      onConnected();
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);

  return <h1>{roomId} 방에 오신 것을 환영합니다!</h1>
}
```
이렇게 `useEffectEvent`를 사용하게되면 문제는 해결됩니다. 다만 Effect 의존성 목록에서 `onConnected`에서 사용한 `theme`는 제거해야한다는 것을 기억하세요.
이벤트 핸들러와 다른 점이 있다면, EffectEvent는 Effect에서 직접 트리거 된다는 것입니다.
`useEffectEvent`를 사용하여 Effect의 반응성과 반응형이어서는 안되는 코드의 연결을 끊어보세요.

### Effect 이벤트로 최근 props와 state 읽기
> 아직 개발중인 기능입니다. useEffect와 같이 실험적인 API이니 주의하세요.

Effect 이벤트는 의존성 린터를 억제하고 싶었을 많은 패턴을 수정하게 합니다.
```jsx
// 페이지 방문을 기록하기 위한 Effect
function Page(url) {
  useEffect(() => {
    logVisit(url);
  }, [url]); // 서로 다른 페이지 방문기록을 기록하기 때문에 의존성으로 추가해야함
  // ...
}

// 방문 기록에 장바구니 물건 개수를 추가하는 경우
function Page({ url }) {
  const { items } = useContext(ShoppingCartContext);
  const numberOfItems = items.length;

  useEffect(() => {
    logVisit(url, numberOfItems);
  }, [url]); // 🔴 React Hook useEffect has a missing dependency: 'numberOfItems'
  // 린터는 'numberOfItems'를 의존성으로 추가하길 원하지만 장바구니에 물건을 넣어 'numberOfItems'가 변경되는 것은 다른 페이지 방문을 의미하지 않음 ==> 페이지 방문은 이벤트
} 


function Page({ url }) {
  const { items } = useContext(ShoppingCartContext);
  const numberOfItems = items.length;
  
  // Effect 이벤트 선언 
  const onVisit = useEffectEvent(visitedUrl => {
    logVisit(visitedUrl, numberOfItems);
  });

  useEffect(() => {
    onVisit(url);
  }, [url]); // ✅ 모든 의존성 선언됨
  // ...
}
```
여기서 `onVisit`은 Effect 이벤트입니다. 코드 내부는 반응형이 아니게됩니다. 그러므로 `numberOfItems`의 변경이 주변 코드를 재실행시킬 걱정없이 사용할 수 있습니다.

Effect 자체는 여전히 반응형입니다. prop인 `url`을 사용해 리랜더링 될때마다 Effect는 재실행되고 `onVisit`이 호출됩니다. 이때 항상 최근의 `numberOfItems`를 읽을 것입니다. 하지만 `numberOfItems`만 변경된다면 어떤 코드도 재실행되지 않습니다.


### Effect 이벤트의 한계

Effect 이벤트는 사용할 수 있는 방법이 매우 제한적입니다.

- **Effect 내부에서만 호출해야합니다.**
- **절대 다른 컴포넌트나 Hook에 전달하면 안됩니다.**