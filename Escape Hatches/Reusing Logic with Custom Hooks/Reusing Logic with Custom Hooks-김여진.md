### custom hook으로 컴포넌트 간 로직 공유, 재사용하기

작명 규칙: `use`로 시작, 그 뒤는 대문자로 시작 / 무슨 일을 하고, 무엇을 props로 받고, 무엇을 반환하는지 알 수 있도록 명명 e.g. useChatRoom(options), useIntersectionObserver(ref, options)

내부에서 하나 이상의 hook을 사용한다면 반드시 함수 앞에 use를 작성해야 함(그 자체로 hook이 됨)
```js
function useAuth() {
  return useContext(Auth);
}
```

custom hook으로 
컴포넌트 코드가 어떻게 그것을 하는지(세부 사항)가 아닌 무엇을 하는지를 나타내는 역할만 하게 함
즉 컴포넌트 코드는 목적만 나타낼 뿐 실행 방법은 나타내지 않게 할 수 있음

하나의 컴포넌트에서 같은 hook을 호출해도 각각의 hook 호출은 완전히 독립되어 있음

custom hook은 컴포넌트가 리렌더링될때마다 다시 실행됨 -> 순수해야함

```js
export function useChatRoom({ serverUrl, roomId, onMessage }) {
  const onReceiveMessage = useEffectEvent(onMessage); // 함수는 의존성 배열에 넣으면 안되고 useEffectEvent로 최신값 트래킹

  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    connection.on('message', (msg) => onReceiveMessage(msg));
    return () => connection.disconnect();
  }, [roomId, serverUrl]);
}

export default function ChatRoom({ roomId, onMessage }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  // ChatRoom 컴포넌트는 커스텀 Hook을 호출해 실행 방법이 아닌 코드의 목적만 나타냄
  useChatRoom({
    serverUrl: serverUrl,
    roomId: roomId,
    onMessage(msg){
      showNotification('New message: ' + msg);
    },
  });

  return (
    <>
      <label>
        Server URL:
        <input value={serverUrl} onChange={e => setServerUrl(e.target.value)} />
      </label>
      <h1>Welcome to the {roomId} room!</h1>
    </>
  );
}
```

```js
// 온라인, 오프라인 상태에 따라 안내 메시지를 출력 (네트워크 상태 변경 감지)

// useOnlineStatus.js
import { useSyncExternalStore } from 'react';
// useSyncExternalStore: React 컴포넌트가 외부 스토어(전역상태관리 라이브러리, 브라우저 API)를 구독하고 최신 상태를 가져오는 데 사용
// SSR을 지원하여 초기 HTML을 생성할때(클라이언트의 데이터를 사용 불가) 서버 전용 값(getServerSnapshot)을 사용함 -> SSR과 CSR의 데이터 일관성 보장_서버 렌더링 시에는 getServerSnapshot을 사용, 클라이언트 렌더링에서는 getSnapshot을 사용

function subscribe(callback) {
  window.addEventListener('online', callback);
  window.addEventListener('offline', callback);
  return () => {
    window.removeEventListener('online', callback);
    window.removeEventListener('offline', callback);
  };
}

export function useOnlineStatus() {
  return useSyncExternalStore( 
    subscribe, // 외부 상태를 구독하기 위한 함수. 구독한 데이터가 변경됐을때만 컴포넌트를 리렌더링함
    // ㄴ 브라우저의 online 및 offline 이벤트를 구독함
    () => navigator.onLine, // 클라이언트에서 현재 상태를 가져오는 함수 (getSnapshot)
    // ㄴ 현재 네트워크 상태 제공
    () => true // 서버 렌더링에서 사용될 값을 가져오는 함수, 서버 렌더링이 없으면 생략 가능 (getServerSnapshot)
    // ㄴ 서버 렌더링시 기본값(true) 반환해 온라인 상태라고 가정
  );
}

// App.js
import { useOnlineStatus } from './useOnlineStatus.js';

function StatusBar() {
  const isOnline = useOnlineStatus(); // 커스텀 훅으로 실행 결과를 받아 사용함
  return <h1>{isOnline ? '✅ Online' : '❌ Disconnected'}</h1>;
}

export default function App() {
  return (
    <>
      <StatusBar />
    </>
  );
}
```
