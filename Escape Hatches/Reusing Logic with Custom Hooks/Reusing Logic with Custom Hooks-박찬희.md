# 커스텀 Hook으로 로직 재사용하기

React 애플리케이션은 여러 컴포넌트로 만들어집니다. 컴포넌트들은 내장되거나 직접 작성한 Hook으로 만들어집니다.
종종 다른 사람들에 의해 만들어진 Hook을 사용했을 것입니다. 하지만 때에 따라 본인이 Hook을 만들어 사용해야할 때도 있습니다.

이때 Hook을 작명하기 위한 규칙으로는 두가지가 있습니다.

1. React 컴포넌트의 이름은 항상 대문자로 시작해야합니다.(ex: `StatusBar`, `SaveButton`)
  또한 React 컴포넌트는 JSX처럼 어떻게 보이는지 React가 알 수 있는 무언가를 반환해야 합니다.
2. Hook의 이름은 `use` 뒤에 대문자로 시작해야합니다. (ex: `useState`(내장 Hook), `useStatus`(커스텀 Hook))
  Hook들은 어떤 값이든 반환할 수 있습니다.
  Hook 내부에는 높은 확률로 다른 Hook을 내부에서 사용하고 있습니다. 반대로 `use`로 시작하지 않는 함수에는 state와 같은 Hook이 사용되지 않는다는 것을 알 수 있습니다.
## 커스텀 Hook

```jsx
import { useState, useEffect } from 'react';

export function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    function handleOnline() {
      setIsOnline(true);
    }
    function handleOffline() {
      setIsOnline(false);
    }
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);
  return isOnline;
}
```

## 커스텀 Hook은 state 자체를 공유하는 것이 아닌 state 저장 로직을 공유하도록 하기

다음의 `Form`컴포넌트를 통해 커스텀 Hook의 로직에 대해 알아봅시다.
```jsx
import { useState } from 'react';

export default function Form() {
  const [firstName, setFirstName] = useState('Mary');
  const [lastName, setLastName] = useState('Poppins');

  function handleFirstNameChange(e) {
    setFirstName(e.target.value);
  }

  function handleLastNameChange(e) {
    setLastName(e.target.value);
  }

  return (
    <>
      <label>
        First name:
        <input value={firstName} onChange={handleFirstNameChange} />
      </label>
      <label>
        Last name:
        <input value={lastName} onChange={handleLastNameChange} />
      </label>
      <p><b>Good morning, {firstName} {lastName}.</b></p>
    </>
  );
}
```
각각의 폼 입력에 반복되는 로직이 있습니다.
1. state가 존재합니다.(`firstName`, `lastName`)
2. 변화를 다루는 함수가 존재합니다.(`handleFirstNameChange`, `handleLastNameChange`)
3. 해당 입력에 대한 `value`와 `onChange`의 속성을 지정하는 JSX가 존재합니다.

이를 활용해 `useFormInput`이라는 커스텀 Hook을 만들어 반복되는 로직을 추출할 수 있습니다.

```jsx
import { useFormInput } from './useFormInput.js';

export default function Form() {
  const firstNameProps = useFormInput('Mary');
  const lastNameProps = useFormInput('Poppins');

  return (
    <>
      <label>
        First name:
        <input {...firstNameProps} />
      </label>
      <label>
        Last name:
        <input {...lastNameProps} />
      </label>
      <p><b>Good morning, {firstNameProps.value} {lastNameProps.value}.</b></p>
    </>
  );
}

//useFormInput.js
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
첫 예제에서 반복되는 로직을 추출하여 `useFormInput` Hook을 만들었습니다. 
여기서 `value`를 통해 서로 다른 두 state를 `useFormInput` 커스텀 Hook을 사용하면 간편하게 state 선언을 할 수 있다는 걸 확인할 수 있습니다.
대신 `useFormInput`은 두 번 호출 됩니다.

여기서 우리는 커스텀 Hook이 state 자체가 아닌 state 저장 로직을 공유하도록 해준다는 것을 알 수 있습니다.
같은 Hook을 호출하더라도 각각의 Hook 호출은 완전히 독립되어 있습니다.

## Hook 사이에 상호작용하는 값 전달하기

커스텀 Hook 안의 코드는 컴포넌트가 재렌더링 될 때마다 다시 돌아갈 겁니다. 이게 바로 커스텀 Hook이 순수해야하는 이유입니다.
커스텀 Hook이 컴포넌트와 함께 재랜더링 된다면, 항상 가장 최신의 props와 state를 전달받을 것입니다. 
이게 무슨 말인지를 알아보기 위해 채팅방 예시를 들어보겠습니다.

```jsx
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';
import { showNotification } from './notifications.js';

export default function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

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
이 컴포넌트는 `serverUrl`나 `roomId`를 변경할 때, Effect가 변화에 반응하며 재동기화합니다. 

Effect 코드를 커스텀 Hook 안에 넣어봅시다.

```jsx
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

`ChatRoom` 컴포넌트가 내부 동작이 어떻게 동작하는지 걱정할 필요 없이 커스텀 Hook을 호출할 수 있게 해줍니다.

```jsx
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
커스텀 Hook으로 Effect를 바꾸면서 매우 간단해졌지만 똑같이 동작합니다.
매번  `ChatRoom`이 리랜더링 될 때마다, Hook에 최신 `roomId`와 `serverUrl` 값을 넘겨줍니다.
이 때문에 리랜더링 이후에 값이 달라지는지 여부에 관계없이 Effect가 재연결하는 이유입니다.


## 언제 커스텀 Hook을 사용해야할까?

모든 자잘한 중복 코드들까지 커스텀 Hook으로 분리할 필요는 없습니다. 
하지만 Effect를 사용하든 하지 않든, 커스텀 Hook 안에 그것을 감싸는 게 좋은지 아닌지 고려해야합니다.
다음 예제를 통해 확인해봅시다. 다음 예제는 두 가지 목록을 보여주는 `ShippingForm` 컴포넌트입니다.

```jsx

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

  // ...
  ```

이 코드는 반복됨에도 불구하고, Effect는 다른 두 가지(도시, 구역)를 동기화하기 때문에 따로 분리하는 것이 옳습니다.
대신 중복되는 부분을 `useData`라는 커스텀 Hook을 통해 추출할 수 있습니다.

```jsx
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
`useData`를 활용해서 `ShippingForm` 컴포넌트 내부의 중복된 Effect들을 교체해봅시다.
```jsx
function ShippingForm({ country }) {
  const cities = useData(`/api/cities?country=${country}`);
  const [city, setCity] = useState(null);
  const areas = useData(city ? `/api/areas?city=${city}` : null);

  // ...
```
가독성이 훨씬 좋아졌습니다. 이제 명확하게 데이터의 흐름을 파악할 수 있습니다.