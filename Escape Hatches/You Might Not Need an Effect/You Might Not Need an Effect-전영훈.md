Effect의 필요 목적은 외부 시스템(non-React widget, network, or the browser DOM.)과 동기화 하기 위함.

# Effect가 불필요한 경우

1. prop이 변경될 때 일부 state 조정하기

=>prop이 변경될 때 전체가 아닌 일부 state만 재설정하거나 조정하고 싶을 때가 있습니다.

```jsx
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selection, setSelection] = useState(null);

  // 🔴 피하세요: Effect에서 prop 변경 시 state 조정하기
  useEffect(() => {
    setSelection(null);
  }, [items]);
  // ...
}
```

이렇게 작성하면, item이 바뀌고, 렌더링이 되어 커밋이 되고, effect가 실행됩니다. effect내부에서 selection의 상태를 바꾸고 재렌더링되는 불필요한 일이 발생합니다.

```jsx
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selectedId, setSelectedId] = useState(null);
  // ✅ 최고예요: 렌더링 중에 모든 것을 계산
  const selection = items.find((item) => item.id === selectedId) ?? null;
  // ...
}
```

이런게 써줌으로써, 렌더링 중에 selection을 계산하여불필요한 재렌더링을 방지할 수 있습니다.

2. POST 요청 보내기

```jsx
function Form() {
  const [firstName, setFirstName] = useState("");
  const [lastName, setLastName] = useState("");

  // ✅ 좋습니다: 컴포넌트가 표시되었으므로 이 로직이 실행되어야 합니다.
  useEffect(() => {
    post("/analytics/event", { eventName: "visit_form" });
  }, []);

  // 🔴 피하세요: Effect 내부의 이벤트별 로직
  const [jsonToSubmit, setJsonToSubmit] = useState(null);
  useEffect(() => {
    if (jsonToSubmit !== null) {
      post("/api/register", jsonToSubmit);
    }
  }, [jsonToSubmit]);

  function handleSubmit(e) {
    e.preventDefault();
    setJsonToSubmit({ firstName, lastName });
  }
  // ...
}
```

해당 코드는 jasonToSubmit의 상태가 변경될 때 마다 자동으로 API 요청을 보내게 되어 불필요한 API 요청이 발생할 수 있음

따라서 굳이 effect 내부로 묶지말고

```jsx
function handleSubmit(e) {
  e.preventDefault();
  // ✅ 좋습니다: 이벤트별 로직은 이벤트 핸들러에 있습니다.
  post("/api/register", { firstName, lastName });
}
```

명백한 트리거조건이 있는 이벤트 함수로 사용하는것이 바람직함

3. 앱 초기화

```jsx
function App() {
  // 🔴 Avoid: Effects with logic that should only ever run once
  useEffect(() => {
    loadDataFromLocalStorage();
    checkAuthToken();
  }, []);
  // ...
}
```

해당 코드는 단 한번 발생하도록 설계한 코드인데, 개발환경에선 버그를 찾기위해 두번 실행되기도함.

```jsx
if (typeof window !== "undefined") {
  // 브라우저에서 실행 중인지 확인합니다.
  // ✅ 앱 로드당 한 번만 실행
  checkAuthToken();
  loadDataFromLocalStorage();
}

function App() {
  // ...
}
```

따라서 해당 코드처럼 사용함으로서 의도치않은 실행을 막을 수 있음.
