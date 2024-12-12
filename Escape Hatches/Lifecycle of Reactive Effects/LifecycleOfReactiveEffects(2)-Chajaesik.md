### 빈 종속성이 있는 effect의 의미

serverUrl와 roomId를 모두 컴포넌트 외부로 이동하면 

```
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

컴포넌트의 관점에서는 빈 [] 의존성 배열은 effect가 컴포넌트가 마운트될 때만 채팅방에 연결되고 컴포넌트가 마운트 해제될 때만 연결이 끊어진다는 것을 의미한다(React는 로직을 테스트하기 위해 개발 단계에서 한 번 더 동기화한다)

 

하지만 effect관점에서 중요한 것은 effect가 동기화를 시작하고 중지하는 작업을 지정한 것이다.

현재 반응형 종속성이 없으므로 serverUrl이 변경되거나 roomId가 변경되어도 userEffect는 재 실행 되지 않을 것이다.

 

### 컴포넌트 본문에서 선언된 모든 변수는 반응형이다.
props나 state만 반응형 값이 아니다.

이를 사용하여 계산하는 값도 반응형이다.

props나 state가 변경되면 컴포넌트가 다시 렌더링되고 그로부터 계산된 값도 변경된다.

이 때문에 effect에서 사용하는 컴포넌트 본문의 모든 변수는 effect 종속성 목록에 있어야 한다.

 

하지만 location.pathname과 같은 변경가능한 값은 종속성이 될 수 없다.

렌더링 도중 변경할  수 있는 데이터를 읽는 것은 렌더링의 순수성을 깨뜨리기 때문에 useSyncExternalStore를 사용하여 외부 변경할 수 있는 값으 읽고 구독해야한다.

 

ref.current나 이 값에서 읽은 것 역시 종속성이 될 수 없다.

ref는 재 렌더링 되지 않고도 무엇을 추적할 수 있는데 변경된 후 렌더링이 트리거 되지 않기에, 반응형 값이 아니다.

따라서 React는 이 값이 변경될 때 effect를 다시 실행하지 않는다.

 

예를 들어, roomId와 serverUrl 모두 반응형이기에 린트 오류이다.
```
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, []); // <-- 여기 무언가 잘못되었습니다!
  ```
반응형은 종속성 배열에 넣어야 effect가 다시 동기화된다.

 

다음과 같이 추가한다.
```
function ChatRoom({ roomId }) { // roomId는 반응형입니다.
  const [serverUrl, setServerUrl] = useState('https://localhost:1234'); // serverUrl는 반응형입니다.
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [serverUrl, roomId]); // ✅ 선언된 모든 종속성
  // ...
}
```

종속성 배열에 들어가는 값은 안정적이여야 한다.

예를 들어, useState에서 반환되는 set 함수와 useRef에서 반환되는 ref 객체는 안정적이며, 다시 렌더링해도 변경되지 않도록 보장된다.

안정된 값은 반응하지 않으므로 목록에서 생략할 수 있다.

따라서 변경되지 않는 값은 포함해도 상관없다.

 

### 다시 동기화하지 않으려는 경우..
랜더링 중에 계산이 되지 않으므로 반응하지 안흔다.
```
function ChatRoom() {
  useEffect(() => {
    const serverUrl = 'https://localhost:1234'; // serverUrl는 반응형이 아닙니다.
    const roomId = 'general'; // roomId는 반응형이 아닙니다.
 ```

종속성을 "선택"할 수 없다.

종속성에는 effect에서 읽은 모든 반응형 값이 포함되어야 한다.

### 만약 무한 루프가 발생한다면.. 린터를 억제해야한다
- effect가 독립적인 동기화 프로세스를 나타내는지 확인
    - effect가 아무것도 동기화하지 않는다면 불필요한 것일 수 있다.
    - 여러 개의 독립적인 것을 동기화하는 경우 분할해야한다.
- props나 state에 "반응"하지 않고 effect를 다시 동기화하지 않고 최신 값을 얻으려면 effect를 반응하는 부분(effect 안)과 반응하지 않는 부분(effect 이벤트라고 하는 것으로 추출할 수 있는 것)을 ㅗ분리하면 된다.
- 객체와 함수를 종속성으로 사용하지 마라
    - 렌더링 중에 오브젝트와 함수를 생성한 다음 eefect에서 읽으면 렌더링할 때마다 오브젝트와 함수가 달라진다
    - 그러면 매번 effect를 다시 동기화해야한다.