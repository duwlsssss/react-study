# 반응형 effect의 생명주기

컴포넌트와 달리 effect는 동기화를 시작하고 나중에 동기화를 중지하는 두 가지 작업만 가능. => 컴포넌트와는 생명주기가 다르다.

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

roomId에 따라 동기화 시작 => 동기화 중지후 cleanUp 발생 (언마운트)
