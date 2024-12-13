## 1. 이벤트 핸들러와 Effect의 역할
- 이벤트 핸들러: 특정 사용자 상호작용에 대한 응답으로 실행됩니다.
- 예: 버튼 클릭, 폼 제출.
- 반응형 값의 변화에 의해 실행되지 않습니다.
``` jsx
function handleSendClick() {
  sendMessage(message); // 클릭 이벤트로만 실행
}
```
- Effect: 특정 상태나 props에 따라 동기화가 필요할 때 실행됩니다.

- 예: 데이터 가져오기, 컴포넌트가 화면에 나타났을 때 서버 연결.
- 반응형 값의 변화에 의해 실행됩니다.
``` jsx
useEffect(() => {
  const connection = createConnection(serverUrl, roomId);
  connection.connect();
  return () => connection.disconnect();
}, [roomId]); // roomId가 변경되면 다시 실행
```
## 2. 반응형 값과 비반응형 로직
- 반응형 값:
    - props, state, 렌더링 중 계산된 값 등 리렌더링 시 변경 가능한 값.
    - Effect는 이를 의존성 배열에 명시해야 함.

- 비반응형 로직:
    - 특정 시점에 실행되지만 반응형 값의 변화에 따라 재실행되지 않아야 하는 로직.
    - 예: 알림 표시, 단순 로깅.
## 3. Effect 이벤트를 사용한 비반응형 로직 분리
- Effect 이벤트:
- Effect 내부에서 실행되는 비반응형 로직을 추출하기 위한 도구.
- 항상 최신 props와 state를 참조하며, 반응형 값의 변화로 재실행되지 않음.
- React의 실험적 API(useEffectEvent)를 사용해 정의.

``` jsx

const onConnected = useEffectEvent(() => {
  showNotification('연결됨!', theme); // theme은 비반응형으로 취급
});

useEffect(() => {
  const connection = createConnection(serverUrl, roomId);
  connection.on('connected', () => {
    onConnected(); // 최신 theme 사용
  });
  connection.connect();
  return () => connection.disconnect();
}, [roomId]); // roomId에만 반응
```
## 4. 이벤트 핸들러와 Effect 이벤트의 차이
- 이벤트 핸들러: 사용자 상호작용에 의해 실행.
    - 예: 버튼 클릭으로 sendMessage 호출.
- Effect 이벤트: Effect 내부에서 실행되며 최신 props와 state를 사용.
    - 예: 채팅 연결 시 showNotification 호출.
## 5. 비반응형 로직과 Effect 이벤트의 이점
- Effect 내부에서 반응형 값(theme)이 변할 때마다 재실행되는 것을 방지.
- 필요하지 않은 재실행을 막아 성능 최적화.
- 명확한 동작:
    - Effect 이벤트는 최신 상태를 참조하므로 잘못된 데이터 참조 방지.
    - 이벤트의 "트리거 이유"를 명확히 전달 가능.
## 6.  예시
- 페이지 방문 기록 예시

- 페이지 URL이 변경될 때 방문 기록을 저장.
- 장바구니 항목 개수는 최신 값으로 기록하지만, 이 값의 변화가 Effect 실행을 유발해서는 안 됨.
``` jsx
const onVisit = useEffectEvent((visitedUrl) => {
  logVisit(visitedUrl, numberOfItems); // numberOfItems는 최신 값 사용
});

useEffect(() => {
  onVisit(url); // URL 변경 시 실행
}, [url]); // URL 변경에만 반응
```
