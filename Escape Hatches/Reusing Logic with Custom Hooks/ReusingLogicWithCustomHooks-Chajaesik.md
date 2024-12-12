커스텀 Hook을 사용하는 이유 : 컴포넌트간 로직을 공유하기 위해

커스텀 Hook은 state 자체가 아닌 state 저장 로직만 공유한다

React는 useState, useContext, useEffect 같이 몇몇 Hook을 제공한다.

우리는 데이터를 fetch하거나 사용자가 온라인 상태인지 확인하는 등 본인만의 Hook을 만들 수 있다.

 

### 학습 내용
- 커스텀 Hook은 무엇이고, 어떻게 만드는가
- 컴포넌트 간 로직 재사용 방법
- 나만의 커스텀 Hook 이름 짓기와 구조 잡기
- 언제 그리고 왜 커스텀 Hook을  추출해야 하는가

### 커스텀 Hook 컴포넌트간 로직 공유하는 방법
예를 들어 아래와 같은 네트워크 값을 State로 관리한다고 가정해보자
```
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    function handleOnline() {
      setIsOnline(true);
    }
    function handleOffline() {
      setIsOnline(false);
    }
 ```

그다음 비슷한 로직으로 isOnline의 값이 false일 때 "저장" 대신 "재 연결 중"을 출력하고싶다고 가정해보자.

 

조금만 생각해본다면 방금 작성한 코드와 동일한 로직임을 다시 사용해야함을 깨달았을 것이다!

따라서 동일한 로직을 재사용하기 위해 Hook으로 만들 것이다.

 

### 컴포넌트로부터 커스텀 Hook 추출하기
방금 작성한 코드를 useOnlineStatus함수로 만들었다고 가정하자
```
function StatusBar() {
  const isOnline = useOnlineStatus(); // 여기서 사용
  return <h1>{isOnline ? '✅ 온라인' : '❌ 연결 안 됨'}</h1>;
}

function SaveButton() {
  const isOnline = useOnlineStatus(); // 여기서 또 사용!

  function handleSaveClick() {
    console.log('✅ 진행사항 저장됨');
  }
  ```
이를 통해 중복되는 로직을 하나의 함수로 만들어 불러오도록 수정했다.

 

중요한건, 두 컴포넌트 내부 코드가 어떻게 그것을 하는지(브라우저 이벤트 구독하기)보다 그들이 무엇을 하려는지 )온라인  state 사용하기)에 대해 설명하고 있다는 점이다.

 

### Hook의 이름은 항상 use로 시작해야한다.
커스텀 훅을 만들 때는 다음의 작명 규칙을 따라야 한다.

- React 컴포넌트의 이름은 항상 대문자로 시작해야 한다
    - React 컴포넌트는 JSX처럼 어떻게 보이는지 작성해야한다.
- Hook의 이름은 use 뒤에 대문자로 시작해야한다.
    - Hook들은 어떤 값이든 반환할 수 있다.
```
// ✅ 좋은 예시 : Hook을 사용하는 Hook
function useAuth() {
  return useContext(Auth);
}
 ```


### 커스텀 Hook은 state 그 자체를 공유하는게 아닌 state 저장 로직을 공유하도록 한다.
아래의 예시를 보자
```
export default function Form() {
  const [firstName, setFirstName] = useState('Mary');
  const [lastName, setLastName] = useState('Poppins');

  function handleFirstNameChange(e) {
    setFirstName(e.target.value);
  }

  function handleLastNameChange(e) {
    setLastName(e.target.value);
  }
```

각각의 폼 입력에 반복되는 로직이 존재한다.

1. state가 존재(firstName와 lastName)
2. 변화를 다루는 함수가 존재(handleFirstName... handleLastName..)
3. 해당 입력에 대한 value와 onChange의 속성을 지정하는 JSX가존재
 

useFormInput 커스텀 Hook을 통해 반복되는 로직을 추출할 수 있다.
```
import { useState } from 'react';

export function useFormInput(initialValue) {
  const [value, setValue] = useState(initialValue);

  function handleChange(e) {
    setValue(e.target.value);
  }

  const inputProps = {
    value: value,
    onChange: handleChange
  };

  return inputProps;
}
 ```

### 아까의 Form에서 사용할 때는..
```
function Form() {
  const firstNameProps = useFormInput('Mary');
  const lastNameProps = useFormInput('Poppins');
  // ...
```

커스텀 Hook은 state 그 자체가 아닌 state 저장 로직을 공유하도록 해준다

따라서 같은 Hook을 호출하더라도 각각의 Hook 호출은 완전히 독립되어 있다.

 

만약 여러 컴폰너트 간 state 자체를 공유할 필요가 있다면, state를 위로 "올려"주면 되겠죠?

 

### Hook 사이에 상호작용하는 값 전달하기
커스텀 Hook이 순수해야 하는 이유 : 커스텀 Hook 안의 코드는 컴포넌트가 재렌더링될 때마다 다시 항상 가장 최신의 props와 state를 전달받을 것이기 때문

 

아래의 코드는 serverUrl나 roomId를 변경할 때 Effect는 변화에 "반응"한다.
```
  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.on('message', (msg) => {
      showNotification('New message: ' + msg);
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]);
``` 

### 이제 Effect 코드를 커스텀 Hook으로 작성해보자!
```
export function useChatRoom({ serverUrl, roomId }) {
  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    connection.on('message', (msg) => {
      showNotification('New message: ' + msg);
    });
    return () => connection.disconnect();
  }, [roomId, serverUrl]);
}
``` 

### 다른 곳에서 사용할 때는
```
import { useState } from 'react';
import { useChatRoom } from './useChatRoom.js';

export default function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useChatRoom({
    roomId: roomId,
    serverUrl: serverUrl
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

매번 ChatRoom가 재렌더링될 때마다 Hook에 최신 roomId와 serverUrl 값을 넘겨준다

이것이 재렌더링 이후에 값이 달라지는지 여부에 관계없이 Effect가 재 연결하는 이유이다.

useState의 결과를 useChatRoom의 입력으로 "입력"하는 것과 동일하다.

### 언제 커스텀 Hook을 사용해야 하는가?
 

모든 자잘한 중복되는 코드들을 커스텀 Hook으로 분리할 필요는 없다.

 

아래의 두 가지 목록을 보여주는 SHippingForm컴포넌트를 살펴보자
```
function ShippingForm({ country }) {
  const [cities, setCities] = useState(null);
  // 이 Effect는 나라별 도시를 불러옵니다.
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
  }, [country]);

  const [city, setCity] = useState(null);
  const [areas, setAreas] = useState(null);
  // 이 Effect 선택된 도시의 구역을 불러옵니다.
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
  }, [city]);
``` 

### 위 코드들은 반복됨에도 불구하고 Effect를 따로 분리하는 것이 옳다

#### 왜냐하면..

그들은 다른 두 가지(도시, 구역)을 동기화하기 때문에 하나의 Effect로 통합시킬 필요가 없다

 

대신 SHippingFOrm 컴포넌트를 useData 라는 커스텀 Hook을 통해 공통된 로직을 추출할 수 있다.
```
function useData(url) {
  const [data, setData] = useState(null);
  useEffect(() => {
    if (url) {
      let ignore = false;
      fetch(url)
        .then(response => response.json())
        .then(json => {
          if (!ignore) {
            setData(json);
          }
        });
      return () => {
        ignore = true;
      };
    }
  }, [url]);
  return data;
}
 ```

### 이제 .SHippingForm 컴포넌트 내부의 Effect들을 useData로 교체할 수 있다.
```
function ShippingForm({ country }) {
  const cities = useData(`/api/cities?country=${country}`);
  const [city, setCity] = useState(null);
  const areas = useData(city ? `/api/areas?city=${city}` : null);
```