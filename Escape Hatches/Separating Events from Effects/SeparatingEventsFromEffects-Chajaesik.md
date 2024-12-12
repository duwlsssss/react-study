### 학습 내용
- 이벤트 핸들러와 Effect 중에 선택하는 방법
- Effect는 반응형이고 이벤트 핸들러는 아닌 이유
- Effect의 코드 일부가 반응형이 아니길 원한다면 해야할 것
- Effect 이벤트의 정의와 Effect에서 추출하는 방법
- Effect 이벤트를 사용해 Effect에서 최근의 props나 state를 읽는 방법
 

### 이벤트 핸들러와 Effect 중에 선택하기
예를 들어.. 채팅방 컴포넌트를 구현한다면 다음의 요구사항을 생각할 수 있다.

1. 채팅방 컴포넌트는 선택된 채팅방에 자동으로 연결해야 한다
2. "전송"버튼을 클릭하면 채팅에 메시지를 전송해야 한다.

이러한 기능을 동작하게 하기 위해 당신은 이벤트 핸들러와 Effect중에 무엇을 사용할 것인가?

### 이벤트 핸들러는 특정 상호작용에 대한 응답으로 실행된다
사용자 관점에서, 메시지는 "전송"버튼이 클릭했을 때 전송되어야 한다.

그러므로 메시지를 전송하는 건 이벤트 핸들러가 처리해야 한다.
```
  function handleSendClick() {
    sendMessage(message);
  }
 ```
### Effect는 동기확가 필요할 때마다 실행된다.
채팅방 컴포넌트는 채팅방과의 연결을 유지해야한다라는 요구사항을 기억하자

 

사용자가 현재 채팅방 화면을 보고 상호작용할 수 있으므로 컴포넌트는 선택된 채팅 서버에 계속 연결되어 있어야 한다.

채팅방 컴포넌트가 앱의 첫 화면이고 상요자가 아무런 상호작용을 하지 않은 경우라 해도 여전히 "연결"되어 있어야 한다.

 

따라서 Effect로 작성한다.
```
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
위 코드같은 effect로 작성하면 사용자가 수행하는 특정 상호작용에 상고나없이 현재 선택된 채팅 서버와 항상 연결됨이 보장된다.

### 반응형 값과 반응형 로직
컴포넌트 본문 내부에 선언된 props, state, 변수를 반응형 값이라고 한다

아래의 예시에서 serverUrl은 반응형이 아니고, roomId와 message는 반응형이다.

반응형 값은 데이터 렌더링 과정에 관여한다.
```
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  // ...
}
```
이러한 반응형 값은 리렌더링으로 변경될 수 있다.

예를 들어 message를 편집하거나 다른 roomId를 선택하는 경우 ..

 

이때 이벤트 핸들러와 Effect는 변화에 다르게 반응한다.

- 이벤트 핸들러 내부의 로직은 반응형이 아니다
    - 사용자가 같은 상호작용(예:클릭)을 반복하지 않는 한 재 실행되지 않는다.
    - 이벤트 핸들러는  변화에 "반응"하지 않으면서 반응형 값을 읽을 수 있다.
- Effect 내부의 로직은 반응형이다.
    - Effect에서 반응형 값을 읽는 경우 그 값을 의존성으로 지정해야 한다.
    - 이를 통해 리렌더링이 그 값을 바꾸는 경우 React가 새로운 값으로 Effect의 로직을 다시 실행한다.
 

### Effect에서 비반응형 로직 추출하기
예를 들어 사용자가 채팅에 연결할 때 알림을 보여주는 상황을 상상해보자
```
function ChatRoom({ roomId, theme }) { // 현재 테마 dark 또는 light
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      showNotification('연결됨!', theme);
    });
    connection.connect();
    return () => {
      connection.disconnect()
    }; // theme는 props로 반응형 값이다. 따라서 의존성으로 선언
  }, [roomId, theme]); // ✅ 모든 의존성 선언됨
  // ...
 ```

### Effect 이벤트 선언하기
 

이 반응형 로직을 Effect에서 추출하려면 useEffectEvent 라는 특수한 Hook을 사용한다.
```
import { useEffect, useEffectEvent } from 'react';

function ChatRoom({ roomId, theme }) {
  const onConnected = useEffectEvent(() => {
    showNotification('연결됨!', theme);
  });
 ```

여기서 onConnected를 Effect 이벤트라 한다.

내부 로직은 항상 반응형이 아니며 항상 props와 state의 "최근 값"을 바라본다.

 

이제 Effect 내부에서 Effect 이벤트인 onConnected를 호출할 수 있다.
```
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
  }, [roomId]); // ✅ 모든 의존성이 선언됨
  // ...
```
Effect 이벤트는 반응형이 아니므로 아까 의존성에 추가했던 theme를 제거해야한다.
 

### Effect 이벤트로 최근 props와 state 읽기
 

예를 들어 페이지 방문 기록을 위한 Effect가 있다고 가정
```
function Page({ url }) {
  useEffect(() => {
    logVisit(url);
  }, [url]); // ✅ 모든 의존성이 선언됨
  // ...
}
 ```

이제, 모든 페이지 방문기록에 장바구니의 물건 개수도 포함하려 한다고 가정
```
function Page({ url }) {
  const { items } = useContext(ShoppingCartContext);
  const numberOfItems = items.length;

  useEffect(() => {
    logVisit(url, numberOfItems);
  }, [url]); // 🔴 React Hook useEffect has a missing dependency: 'numberOfItems'
  // ...
}
```
Effect 내부에서 numberOfItems를 사용했으므로 이를 의존성에 추가해야 하지만

logVisit 호출이 numberOfItems에 반응하지 않기를 원한다

왜냐하면 사용자가 장바구니에 무엇을 넣어 numberOfItems가 변경되는 것이 사용자가 페이지를 다시 방문했음을 의미하지 않기 때문이다.

 

장바구니에 아이템이 추가되는 것과 페이지 방분믄 "별개"이다.

따라서 이를 둘로 나눈다
```
function Page({ url }) {
  const { items } = useContext(ShoppingCartContext);
  const numberOfItems = items.length;

  const onVisit = useEffectEvent(visitedUrl => {
    logVisit(visitedUrl, numberOfItems);
  });

  useEffect(() => {
    onVisit(url);
  }, [url]); // ✅ 모든 의존성 선언됨
  // ...
}
```
여기서, onVisit은 useEffectEvent로 그 내부의 코드는 반응형이 아니다

따라서 numberOfItems(또는 다른 반응형 값)의 변경이 주변 코드를 재실행 시킬 걱정없이 사용할 수 있다.

 

반면에 Effect 자체는 여전히 반응형이다.

prop인 url을 사용하여 값이 변경될 때마다 다른 url로 리렌더링된다.

이 과정에서 Effect가 재실행되면서 onVisit이 호출될 것이다.

 

결과적으로 prop인 url 변경될 때마다 logVisit을 호출할 것이고, 항상 최근의 numberOfItems를 읽을 것이다.

하지만 numberOfItems 혼자만 변경되면 어떠한 코드도 재실행되지 않는다.

 

### useEffectEvent는 사용할 수 있는 방법이 매우 제한적이다.
- Effect 내부에서만 호출
- 절대로 다른 컴포넌트나 Hook에 전달하지 말아야 한다.
```
  const onTick = useEffectEvent(() => {
    setCount(count + 1);
  });

  useTimer(onTick, 1000); // 🔴 금지: Effect 이벤트 전달하기
  
  // 대신 항상 자신을 사용하는 Effect 바로 근처에 선언
  
  function useTimer(callback, delay) {
  const onTick = useEffectEvent(() => {
    callback();
  });
    useEffect(() => {
    const id = setInterval(() => {
      onTick(); // ✅ 바람직함: Effect 내부에서 지역적으로만 호출됨
    }, delay);
 ```

EffectEvent는 비반응형임에 주의하라.