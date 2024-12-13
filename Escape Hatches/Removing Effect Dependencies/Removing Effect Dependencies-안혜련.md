## 1. Effect 의존성이란?
- Effect 의존성은 useEffect 내부에서 사용하는 반응형 값(props, state 등)을 가리킴.
- React 린터는 Effect가 사용하는 모든 반응형 값을 의존성으로 선언했는지 확인함.
- 의존성 목록이 정확하면 컴포넌트가 최신 상태를 유지함.
## 2. 의존성 문제 해결 방법
- Effect는 사용하는 모든 반응형 값을 의존성 목록에 포함해야 함.
- 반응형 값은 컴포넌트가 다시 렌더링될 때 값이 바뀔 수 있는 모든 변수:
props, state, 컴포넌트 내부에서 선언된 변수/함수 등.
- 불필요한 의존성 해결 방법
    1. Effect 내부로 로직 이동:
        - 반응형 값에 의존하는 객체나 함수를 useEffect 내부로 이동하면 더 이상 의존성이 아님.
``` jsx
useEffect(() => {
  const options = { serverUrl: 'https://localhost:1234', roomId };
  const connection = createConnection(options);
  connection.connect();
  return () => connection.disconnect();
}, [roomId]);
```

2. 객체/함수 외부로 이동
    - 반응형 값과 무관한 객체나 함수는 컴포넌트 외부로 이동.


``` jsx

const options = { serverUrl: 'https://localhost:1234', roomId: 'music' };
useEffect(() => {
  const connection = createConnection(options);
  connection.connect();
  return () => connection.disconnect();
}, []); // ✅ 의존성 필요 없음
```

3. 업데이터 함수 사용:

    - 이전 상태를 기반으로 새로운 값을 설정할 경우, 상태 업데이트 함수로 해결.
``` jsx
connection.on('message', (message) => {
  setMessages((prevMessages) => [...prevMessages, message]);
});
```

4. Effect 이벤트 활용:

    - 최신 값을 읽지만 변경에 반응하지 않는 로직은 useEffectEvent를 사용해 분리.
``` jsx

const onMessage = useEffectEvent((message) => {
  if (!isMuted) playSound();
});
```

## 3. 의존성 문제의 원인과 해결책
### 무한 루프 문제
-> 객체/함수 의존성이 매 렌더링마다 새로 생성됨.
    
- 해결
    - 객체/함수를 Effect 내부로 이동.

    -객체/함수를 컴포넌트 외부로 이동(반응형 값에 의존하지 않을 경우).

    -객체의 속성이나 결과를 직접 의존성에 추가.

### 관련 없는 작업을 하나의 Effect에 포함
-> 의존성이 불필요하게 결합되어 Effect가 예상치 않게 재실행.
    - 해결
    - 관련 없는 작업은 독립적인 Effect로 분리.
``` jsx

useEffect(() => {
  fetchCities(country).then(setCities);
}, [country]);

useEffect(() => {
  fetchAreas(city).then(setAreas);
}, [city]);
```

### Effect에서 "반응"하지 않고 값 읽기
- 문제: 읽기만 해야 하는 값을 의존성에 포함해 불필요한 재실행 발생.
    - 해결
    - useEffectEvent를 사용해 최신 값을 읽되 의존성에서 제외.
``` jsx

const onMessage = useEffectEvent((message) => {
  if (!isMuted) playSound();
});
```

## 4. 객체와 함수 의존성을 피하는 이유
- 자바스크립트에서 새로 생성된 객체/함수는 매번 새로운 것으로 간주됨.
- 이로 인해 의도치 않게 Effect가 반복 실행되거나 무한 루프 발생.
- 해결 방법
1. 컴포넌트 외부로 이동
``` jsx
const options = { serverUrl: 'https://localhost:1234', roomId: 'music' };
```
2. Effect 내부로 이동
``` jsx

useEffect(() => {
  const options = { serverUrl: serverUrl, roomId };
}, [roomId]);
```

3. 객체에서 원시 값만 읽기
``` jsx

const { roomId, serverUrl } = options;
useEffect(() => {
  const connection = createConnection({ roomId, serverUrl });
}, [roomId, serverUrl]);
```

## 5. 이벤트 핸들러와 Effect의 구분
### 이벤트 핸들러 
- 특정 사용자 상호작용(버튼 클릭 등)에 반응.
- 로직이 반응형일 필요 없음.
``` jsx

function handleSubmit() {
  post('/api/register');
  showNotification('Registered!');
}
```
### Effect:

- 상태 또는 props의 변화에 따라 자동으로 실행.
``` jsx

useEffect(() => {
  fetchCities(country).then(setCities);
}, [country]);
```