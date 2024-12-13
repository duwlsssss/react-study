### Effect에서 이벤트 분리하기

이벤트는 상호작용에 의하여 발생합니다.
Effect는 동기화가 필요할 때마다 실행됩니다.

=> 이벤트 핸들러는 버튼 클릭과 같이 항상 “수동으로” 트리거 되지만, Effect는 동기화 유지에 필요한 만큼 자주 실행 및 재실행되기 때문에 “자동으로” 트리거 됩니다

```jsx
function ChatRoom({ roomId }) {
  // ...
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

해당코드 발생시 마운트시 또는 roomId가 바뀔 때마다 포넌트는 선택된 채팅 서버에 계속 연결되어있음. (언마운트시에만 disconnect)

## 반응형 값과 반응형 로직

` sendMessage(message);`

해당 코드는 message에 대한 변경일뿐, 메세지를 전송하고자 하는 로직이 아니므로, 이벤트 핸들러에 속함.

```jsx
const connection = createConnection(serverUrl, roomId);
connection.connect();
```

해당 코드는 roomId가 바뀜으로써 다른 방에 연결이 됨. 즉 반응형이기에, effect에 속함.
