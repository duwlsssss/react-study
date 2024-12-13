### Effect의 의존성 제거하기

Effect의 코드에서 사용되는 모든 반응형 값은 의존성 목록에 선언되어야 합니다.

```jsx
const serverUrl = "https://localhost:1234";

function ChatRoom({ roomId }) {
  // This is a reactive value
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // This Effect reads that reactive value
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ So you must specify that reactive value as a dependency of your Effect
  // ...
}
```

만약 반응형 값을 의존성 목록에서 제거하면, 린터가 경고할 것입니다.
그럼에도 불구하고 의존성 목록에서 빼고싶다?

const roomId = 'music'; 이런 방식으로 고정값을 두면 더이상 반응형 값이 아니기에, 의존성 목록에서 제거할 수 있게됩니다.

=> 즉 반응형 값이 아닌 이상, 특별한 이유가 아니고선 의존성 배열에 포함시킬 필요가 없다는거죠.

## 불필요한 의존성 제거하기

# effect가 관련 없는 작업을 수행시

```jsx
function ShippingForm({ country }) {
const [cities, setCities] = useState(null);
const [city, setCity] = useState(null);
const [areas, setAreas] = useState(null);

  useEffect(() => {
    let ignore = false;
    fetch(`/api/cities?country=${country}`)
      .then(response => response.json())
      .then(json => {
        if (!ignore) {
          setCities(json);
        }
      });
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
  }, [country, city]); // ✅ All dependencies declared

```

해당 코드와 같이 쓰면 country가 변경될때 setCities가 동작하여 city가 바뀌고 의존성배열에따라 다시 country가 불러오는 불필요한 동작이 일어납니다.

```jsx
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

이렇게 분리해주면, 각각 독립적인 동기화 프로세스를 가져, 불필요한 fetching을 막을 수 있습니다.

다른 불필요한 예시입니다.

```jsx
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      setMessages([...messages, receivedMessage]);
    });
    return () => connection.disconnect();
  }, [roomId, messages]); // ✅ All dependencies declared
```

해당 코드의 경우, messages를 의존성으로 갖고있어, message를 새로 받을때마다 effect가 발생하여 채팅을 다시연결하는 불필요한 동작이 일어납니다. 이를 막기위해,

```jsx
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      setMessages(msgs => [...msgs, receivedMessage]);
    });
    return () => connection.disconnect();
  }, [roomId]); // ✅ All dependencies declared
```

message는 의존성 배열에서 빼주고, 기존의 message상태의 변경은 업데이트 함수로 실시간으로 messages의 상태를 변경할 수 있도록 할 수 있습니다.
