Effect를 작성하면 린터는 Effect의 의존성 목록에 Effect가 읽는 모든 반응형 값(예를 들어 props 및 State)을 포함했는지 확인한다.

 

### 학습목표
- Effect 의존성 무한 루프 수정하는 방법
- 의존성을 제거하고자 할 때 해야 할 일
- Effecct에 "반응"하지 않고  Effect에서 값을 읽는 방법
- 객체와 함수 의존성을 피하는 방법과 이유
- 의존성 린터를 억제하는 것이 위험한 이유와 대신 할 수 있는 일

### 의존성을 제거하려면 의존성이 아님을 증명해야한다
 
Effect의 의존성을 "선택 할 수 없다"는 점에 유의하라

이전 포스트에서 우리는 Effect안에서 사용되는 모든 "반응형  값"은 의존성 목록에 선언되어야 한다.

반응형 값  ; props와 컴푸넌트 내부에서 선언된 모든 변수 및 함수가 포함

 

따라서, 의존성을 제거하려면 해당 컴포넌트가 의존성이 될 필요가 없다는 것(반응형이 아님)을 린트에게 증명해야한다
```
const serverUrl = 'https://localhost:1234';
const roomId = 'music'; // Not a reactive value anymore // 반응형 값이 아님!

function ChatRoom() {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, []); // ✅ All dependencies declared 처음 마운트될 때만 사용됨
  // ...
}
 ```

### 의존성을 변경하려면 코드를 변경하라
- 먼저 Effect의 코드 또는 반응형 값 선언 방식을 변경한다
- 그런 다음, 변경한 코드에 맞게 의존성을 조정한다
- 의조넝 목록이 마음에 들지 않으면 첫 번째 단계로 돌아간다(그리고 코드를 다시 변경)

의존성 목록 = Effect의 코드에서 사용하는 모든 반응형 값의 목록임을 기억하자

 

### 불 필요한 의존성 제거하기
 

이전 포스트에서 사용자 관점과 useEffect 관점에서 이벤트 핸들러를 쓸지 useEffect를 사용할지를 결정하는 방법에 대해 배웠었다.

 

그렇다면 아래 코드를 보고 "이벤트 핸들러로 옮겨야 할까?"에 대한 답을 스스로 해보기 바란다.
```
function Form() {
  const [submitted, setSubmitted] = useState(false);

  useEffect(() => {
    if (submitted) {
      // 🔴 Avoid: Event-specific logic inside an Effect
      post('/api/register');
      showNotification('Successfully registered!');
    }
  }, [submitted]);
  
    function handleSubmit() {
    setSubmitted(true);
  }
  ```
제출할 때 submiteed State 변수의 값을 true로 설정한다.

POST 요청을 보내고 알림을 표시하는 로직을 true가 될 때 "반응"하는 useEffect안에 넣었다.

 

여기에 현재 테마에 따라 알림 메시지의 스타일을 지정하고 싶어 추가한다.
```
function Form() {
  const [submitted, setSubmitted] = useState(false);
  const theme = useContext(ThemeContext);

  useEffect(() => {
    if (submitted) {
      // 🔴 Avoid: Event-specific logic inside an Effect
      post('/api/register');
      showNotification('Successfully registered!', theme);
    }
  }, [submitted, theme]);
``` 

### !!!!! 에러 발생 !!!!!

 

theme가 변경되고 Effect가 다시 실행되어 동일한 알림이 다시 표시되기 때문이다.

 

여기서 문제는 애초에 Effect로 처리한 것이다.

POST요청을 보내는 것은 사용자가 "제출"할 때인데, 이것은 특정 상호작용인 폼 제출에 대한 응답이다.

특정 상호작용에 대한 응답으로 일부 코드를 실행하려면 해당 로직을 해당 이벤트 핸들러에 직접 넣어야 한다.
```
function Form() {
  const theme = useContext(ThemeContext);

  function handleSubmit() {
    // ✅ Good: Event-specific logic is called from event handlers
    post('/api/register');
    showNotification('Successfully registered!', theme);
  }  

  // ...
}
```
이제 코드가 이벤트 핸들러에 존재하므로, 사용자가 폼을 제출할 때만 실행된다(반응형 코드가 아니므로)

 
## Effect가 관련 없는 여러가지 작업을 수행하나요?
```
    // 🔴 Avoid: A single Effect synchronizes two independent processes
    if (city) {
      fetch(`/api/areas?city=${city}`)
        .then(response => response.json())
        .then(json => {
          if (!ignore) {
            setAreas(json);
          }
        });
    }
    return () => {
      ignore = true;
    };
  }, [country, city]);
  ```
 Effect가 city의 값을 사용하므로 의존성 배열에 추가해야 한다.

이로인해 사용자가 다른 도시를 선택하면 Effect가 다시 실행되어 fetchCities(country)를 또 호출한다.

결과적으로 불필요하게 도시 목록을 여러번 가져오는 것이다.

 

위 코드의 문제점은 서로 관련이 없는 두 가지를 동기화하고 있는 것이다.

country props를 기반으로 cities State를 네트워크에 동기화하려 한다.
city State를 기반으로 areas State를 네트워크에 동기화하려고 한다.
이전 포스트에서 했듯, 두 개의 Effect로 분할한다.
```
function ShippingForm({ country }) {
  const [cities, setCities] = useState(null);
  useEffect(() => {
    let ignore = false;
    fetch(`/api/cities?country=${country}`)
      .then(response => response.json())
      .then(json => {
        if (!ignore) {
          setCities(json);
        }
      });
    return () => {
      ignore = true;
    };
  }, [country]); // ✅ All dependencies declared

  const [city, setCity] = useState(null);
  const [areas, setAreas] = useState(null);
  useEffect(() => {
    if (city) {
      let ignore = false;
      fetch(`/api/areas?city=${city}`)
        .then(response => response.json())
        .then(json => {
          if (!ignore) {
            setAreas(json);
          }
        });
      return () => {
        ignore = true;
      };
    }
  }, [city]); // ✅ All dependencies declared
 ```

이제 첫번째 Effect는 country가 변경될 때만 다시 실행되고,

두번째 Effect는 city가 변경될 때 다시 실행된다.

각 Effect는 독립적인 동기화 프로세스를 나타내야한다를 기억하라

## 만약 중복이 걱정된다면 반복되는 로직을 커스텀 훅으로 만들면된다!
### 다음 State를 계산하기 위해 어떤 State를 읽고 있는가?
 

아래 Effect는 새 메시지가 도착할 때마다 새로 생성된 배열로 messages State 변수를 업데이트 한다.
```
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      setMessages([...messages, receivedMessage]); //스프레드 연산자로 "순수"한 배열 생성
    });
    return () => connection.disconnect();
  }, [roomId, messages]); // ✅ All dependencies declared
  ```
messages 변수를 사용하여 모든 기존 메시지로 시작하는 새배열 을 생성하고 마지막에 새 메시지를 추가한다.

 

## !!!!하지만!!!! messages를 의존성으로 만들면 문제가 발생한다.

1. 메시지를 수신할 때마다 setMessages()는 컴포넌트가 수신된 메시지를 포함하는 새 messages 배열로 재렌더링하도록 한다.

2. 하지만 이 Effect는 이제 messages에 따라 달라지므로 Effect도 다시 동기화된다

따라서 새 메시지가 올 때마다 채팅이 다시 연결된다

 

따라서 Effect 내에서 messages를 읽는 것이 아닌 업데이터 함수를 이용해 setMessages에 전달하자

 
```
 ... 
      setMessages(msgs => [...msgs, receivedMessage]); 업데이터 함수
    });
    return () => connection.disconnect();
  }, [roomId]); // ✅ All dependencies declared
 ```

React는 업데이터 함수를 대기열에 넣고 다음 렌더링 중에 msgs 인수를 제공한다.

 

 

### 일부 반응형 값이 의도치 않게 변경된다면 ..
```
function ChatRoom({ roomId }) {
  // ...
  const options = {
    serverUrl: serverUrl,
    roomId: roomId
  };

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [options]); // ✅ All dependencies declared
 ```

options객체는 컴포넌트 본문에서 선언되므로 반응형 값(rommId를 props로 받기 때문)이다.

Effect 내에서 이와 같은 반응형 값을 읽으면 의존성으로 선언한다.

 

 

하지만 이것은 매번 메시지를 입력할 때마다 채팅방을 닷 ㅣ연결해줘야한다.

이런 문제는 객체와 함수에만 영향을 준다 

자바스크립트에서는 새로 생성된 객체와 함수가 다른 모든 객체와 구별되는 것으로 간주되기 때문이다.
```

// During the first render
const options1 = { serverUrl: 'https://localhost:1234', roomId: 'music' };

// During the next render
const options2 = { serverUrl: 'https://localhost:1234', roomId: 'music' };

// These are two different objects!
console.log(Object.is(options1, options2)); // false
 
```
### 객체에서 원시 값 읽기
 

 props에서 객체를 받을 수도 있다.

 
```
function ChatRoom({ options }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [options]); // ✅ All dependencies declared
  // ...
 ```

이때 부모 컴포넌트에선 아래와 같이 "내려"주겠죠?
```
<ChatRoom
  roomId={roomId}
  options={{
    serverUrl: serverUrl,
    roomId: roomId
  }}
/>
```
이렇게 작성하면 부모 컴포넌트가 재렌더링될 때마다 Effect가 다시 연결된다 

 

### 이러한 문제를 해결하려면 Effeect 외부의 객체에서 정보를 읽고 객체 및 함수 의존성을 피하자!

 
```
function ChatRoom({ options }) {
  const [message, setMessage] = useState('');

  const { roomId, serverUrl } = options; // 구조분해할당
  useEffect(() => {
    const connection = createConnection({
      roomId: roomId,
      serverUrl: serverUrl
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]); // ✅ All dependencies declared
  // ...
 ```

즉, Effect 외부의 객체에서 일부 값을 읽은 다음 Effect 내부에 동일한 값을 가진 객체를 만들면 된다.

 

추가적으로, 의존성을 만들지 않으려면(재렌더링할 때 다시 연결되는 것을 방지하려면)
Effect 외부에서 호출하면 객체가 아니며 Effect 내부에서 읽을 수 있는 roomId 및 serverUrl의 값을 얻을 수 있다.

 
```
function ChatRoom({ getOptions }) {
  const [message, setMessage] = useState('');

  const { roomId, serverUrl } = getOptions();
  useEffect(() => {
    const connection = createConnection({
      roomId: roomId,
      serverUrl: serverUrl
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]); // ✅ All dependencies declared
  // ...
 ```

함수가 이벤트 핸들러이지만 변경 사항으로 인해 Effect가 다시 동기화되는 것을 원치 않는경우

Effect 이벤트로 함수를 감싸면 된다!