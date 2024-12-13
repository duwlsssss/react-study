# 커스텀 훅

```jsx
function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    function handleOnline() {
      setIsOnline(true);
    }
    function handleOffline() {
      setIsOnline(false);
    }
    window.addEventListener("online", handleOnline);
    window.addEventListener("offline", handleOffline);
    return () => {
      window.removeEventListener("online", handleOnline);
      window.removeEventListener("offline", handleOffline);
    };
  }, []);
  return isOnline;
}
```

커스텀 훅의 사용 예시입니다.

1. use로 명명을 시작하되, use 뒤의 이름은 대문자입니다.
2. 내부에 훅을 사용하고 있습니다.
3. 내부 로직처럼 재사용 가능성이 있으며, 긴 코드 라인을 차지할 가능성이 있습니다. (가독성을 낮춤)
4. return으로 코드를 반환하며, 훅을 사용하는 컴포넌트에선 해당 값만 뽑아서 사용할 수 있게 됩니다.

## 커스텀훅은 state 저장 로직을 공유하도록 합니다.

```jsx
import { useState } from "react";

export default function Form() {
  const [firstName, setFirstName] = useState("Mary");
  const [lastName, setLastName] = useState("Poppins");

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
      <p>
        <b>
          Good morning, {firstName} {lastName}.
        </b>
      </p>
    </>
  );
}
```

해당 코드의 경우, state저장 로직이 같은 두 state가 있는데, 이 비슷한 함수를 커스텀 훅으로

```jsx
import { useState } from "react";

export function useFormInput(initialValue) {
  const [value, setValue] = useState(initialValue);

  function handleChange(e) {
    setValue(e.target.value);
  }

  const inputProps = {
    value: value,
    onChange: handleChange,
  };

  return inputProps;
}
```

이렇게 빼주고 외부엔  
`const firstNameProps = useFormInput('Mary');`
`const lastNameProps = useFormInput('Poppins');`
이렇게 써주며, 함수를 하나로 묶을 수 있게되고 렌더링 로직만 존재할 수 있게 해줄 수 있습니다.

같은 Hook을 호출하더라도 각각의 Hook 호출은 완전히 독립되어 있어 가능한 일입니다.

또한, 커스텀 훅 내부의 코드는 컴포넌트가 재렌더링 될 때마다 다시 실행됩니다 (커스텀 훅이 순수해야하는 이유)

## 언제 커스텀 훅을 써야할까!

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
```

해당 코드처럼, 반복되는 로직이지만 effect를 각각 따로 써야할때! ( 해당코드에선, 각각 의존성 배열에서 다른 값을 동기화해서)

이 때 우리는 해당 로직을 커스텀훅으로 묶어줌으로서, 코드의 가독성을 높힐 수 있어요.

```jsx
function useData(url) {
  const [data, setData] = useState(null);
  useEffect(() => {
    if (url) {
      let ignore = false;
      fetch(url)
        .then((response) => response.json())
        .then((json) => {
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

해당 커스텀훅을 만들어서,

```jsx
function ShippingForm({ country }) {
  const cities = useData(`/api/cities?country=${country}`);
  const [city, setCity] = useState(null);
  const areas = useData(city ? `/api/areas?city=${city}` : null);
```

이렇게요!
