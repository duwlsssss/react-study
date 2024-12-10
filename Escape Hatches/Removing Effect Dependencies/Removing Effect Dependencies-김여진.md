### Effect에서 불필요한 의존성 제거하는 방법

Effect의 의존성은 선택하는 게 아님. 코드에 따라 반응형값이 정해지고 이들의 최신값을 사용해 동기화를 하기 위해 모든 반응형값을 포함해야함
하지만 불필요한 의존성으로 Effect가 너무 자주 실행되거나, 무한 루프를 생성하기도 함

1. Effect가 아닌 이벤트 핸들러로 옮겨야 하는지 확인하기
특정 상호작용에 대한 응답 로직은 이벤트 핸들러에 넣기 e.g. 폼제출에 대한 응답으로 알림표시하려면 이벤트 핸들러에

2. Effect가 관련 없는 여러 작업을 수행하면 다른 Effect로 분리하기, 각 Effect는 동기화해야 하는 반응형값만 의존성으로 가짐 

3. 객체와 함수를 Effect의 의존성으로 사용하지 않기
```js
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      // 🔴 Bad
      setMessages([...messages, receivedMessage]); 
    });
  }, [roomId, messages]); // messages는 참조값을 가짐. setMessages로 값을 변경하면 Effect가 재실행될 것임 -> 무한루프
    // 🟢 Good 업데이터 함수를 사용해 messages를 의존성 배열에서 제거 가능
    connection.on('message', (receivedMessage) => {
      setMessages(msgs => [...msgs, receivedMessage]); 
    });
  }, [roomId]);
  // ...
```
```js
// 변경되지 않는 객체, 함수면 컴포넌트 외부로 이동하기 -> 의존성 배열을 비움
const options = {
  serverUrl: 'https://localhost:1234',
  roomId: 'music'
};

function createOptions() {
  return {
    serverUrl: 'https://localhost:1234',
    roomId: 'music'
  };
}

function ChatRoom() {
  const [message, setMessage] = useState('');

  useEffect(() => {
    // 변경되는 객체나 함수는 Effect 내부에 적기 -> 의존성 배열에서 객체, 함수 뺄 수 있음
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); 
  // ...
```

```js
// 
<ChatRoom
  options={{
    serverUrl: serverUrl,
    roomId: roomId
  }}
/>

// 부모에서 참조값을 전달하는 경우
function ChatRoom({ options }) {
  const [message, setMessage] = useState('');

  const { roomId, serverUrl } = options; // 의존성 배열에 options를 쓰면 암됨. 부모 리렌더링시 새 참조값이 생성됨
  // Effect 외부에서 객체에서 일부 값을 읽은 다음 Effect 내부에 동일한 값을 가진 객체를 만듦
  useEffect(() => {
    const connection = createConnection({
      roomId: roomId,
      serverUrl: serverUrl
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]); // options.roomId 또는 options.serverUrl가 실제로 바뀔 때만 채팅에 연결됨
  // ...

// 부모를 수정할 수 있으면 이게 제일 좋음
<ChatRoom
  roomId={roomId}
  serverUrl={serverUrl}
/>


<ChatRoom
  roomId={roomId}
  serverUrl={serverUrl}
  onMessage={msg => {
    showNotification('New message: ' + msg, isDark ? 'dark' : 'light');
  }}
/>

// 부모에서 참조값을 전달하는 경우
function ChatRoom({ roomId, serverUrl, onMessage }) {
  const onRecieveMessage = useEffectEvent(onMessage); // 반응형값인 최신 onMessage함수를 읽을 수 있음 

  useEffect(() => {
    const connection = createConnection({
      serverUrl: serverUrl,
      roomId: roomId
    });
    connection.on('message', (msg) => onReceiveMessage(msg));
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]);
  // ...
```

4. Effect에서 반응하지 않고 읽기만 해야하는 로직은 useEffectEvent로 옮기기
```js
<ChatRoom
  roomId={roomId}
  onReceiveMessage={receivedMessage => {
    // ...
  }}
/>

// 부모로부터 onReceiveMessage함수를 받음 -> onReceiveMessage를 의존성 배열에 넣으면 부모 리렌더링시 Effect가 다시 동기화됨 
function ChatRoom({ roomId, onReceiveMessage }) {
  const [messages, setMessages] = useState([]);

  // useEffectEvent로 감싸 반응형값 onReceiveMessage를 읽기만 함
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
  }, [roomId]); 
  // ...
```



