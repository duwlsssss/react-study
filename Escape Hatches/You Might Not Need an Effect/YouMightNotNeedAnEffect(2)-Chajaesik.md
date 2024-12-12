### POST 요청 보내기
아래 Form 컴포넌트는 두 가지 종류의 POST 요청을 전송한다.

마운트될 때 analytics 이벤트를 보낸다.

폼을 작성하고 Submit 버튼을 클릭하면 /api/register 엔드포인트로 POST 요청을 보낸다.

 
```
function Form() {
	const [firstName, setFirstName] = useState('');
    const [lastName, setLastName] = useState('');
    
    // ✅ 좋습니다: 컴포넌트가 표시되었으므로 이 로직이 실행되어야 합니다.
    useEffect(() => {
    	post('/analytics/event', { eventName: 'visit_form' });
    }, []);
    
   // 🔴 피하세요: Effect 내부의 이벤트별 로직
  const [jsonToSubmit, setJsonToSubmit] = useState(null);
  useEffect(() => {
    if (jsonToSubmit !== null) {
      post('/api/register', jsonToSubmit);
    }
  }, [jsonToSubmit]);
 ```

그러나 /api/register POST 요청은 표시되는 폼으로 인해 발생하는 것이 아닌, 사용자가 벝느을 누르는 특정 시점이다.

 
```
function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');

  // ✅ 좋습니다: 컴포넌트가 표시되었으므로 이 로직이 실행됩니다.
  useEffect(() => {
    post('/analytics/event', { eventName: 'visit_form' });
  }, []);

  function handleSubmit(e) {
    e.preventDefault();
    // ✅ 좋습니다: 이벤트별 로직은 이벤트 핸들러에 있습니다.
    post('/api/register', { firstName, lastName });
  }
  // ...
}
 ```

### 연쇄 계산
때때로 각각 state에 따라 각각 state를 조정하는 Effect를 체이닝 하고 싶을 떄.. 

 
```
// 🔴 피하세요: 서로를 트리거하기 위해서만 state를 조정하는 Effect 체인
  useEffect(() => {
    if (card !== null && card.gold) {
      setGoldCardCount(c => c + 1);
    }
  }, [card]);

  useEffect(() => {
    if (goldCardCount > 3) {
      setRound(r => r + 1)
      setGoldCardCount(0);
    }
  }, [goldCardCount]);
 ```

### 문제
1. 매우 비효율적이다. (setCard → 렌더링 → setGoldCardCount → 렌더링 → setRound → 렌더링 → setIsGameOver → 렌더링)

2. 코드가 발전함에 따라 작성한 "체인"이 새로운 요구 사항에 맞지 않는 경우가 발생할 수 있다.

 

따라서 렌더링 중에 가능한 것을 계산하고 이벤트 핸들러에서 state를 조정하는 것이 좋다
```
function Game() {
  const [card, setCard] = useState(null);
  const [goldCardCount, setGoldCardCount] = useState(0);
  const [round, setRound] = useState(1);

  // ✅ 렌더링 중에 가능한 것을 계산합니다.
  const isGameOver = round > 5;

  function handlePlaceCard(nextCard) {
    if (isGameOver) {
      throw Error('Game already ended.');
    }

    // ✅ 이벤트 핸들러에서 다음 state를 모두 계산합니다.
    setCard(nextCard);
    if (nextCard.gold) {
      if (goldCardCount <= 3) {
        setGoldCardCount(goldCardCount + 1);
      } else {
        setGoldCardCount(0);
        setRound(round + 1); // state는 스냅샷처럼 동작한다.
        if (round === 5) {
          alert('Good game!');
        }
      }
    }
  }
 
```
 

### 애플리케이션 초기화
```
function App() {
  // 🔴 피하세요: 한 번만 실행되어야 하는 로직이 포함된 Effect
  useEffect(() => {
    loadDataFromLocalStorage();
    checkAuthToken();
  }, []);
  // ...
}
```
#### 일부 로직이 컴포넌트 마운트당 한 번이 아니라 앱 로드당 한 번 실행되어야 하는 경우 최상위 변수를 추가하여 이미 실행되었는지를 추적하자

```
let didInit = false;

function App() {
  useEffect(() => {
    if (!didInit) {
      didInit = true;
      // ✅ 앱 로드당 한 번만 실행
      loadDataFromLocalStorage();
      checkAuthToken();
    }
  }, []);
  // ...
}
 ```

### 또는 모듈 초기화 중이나 앱이 렌더링 되기 전에 실행할 수도 있다.

```
if (typeof window !== 'undefined') { // 브라우저에서 실행 중인지 확인합니다.
   // ✅ 앱 로드당 한 번만 실행
  checkAuthToken();
  loadDataFromLocalStorage();
}

function App() {
  // ...
}
```
컴포넌트를 import할 때 최상위 레벨의 코드는 렌더링 되지 않더라도 한 번 실행된다.

 

### State 변경을 부모 컴포넌트에게 알리기
```
function Toggle({ onChange }) {
  const [isOn, setIsOn] = useState(false);

  // 🔴 피하세요: onChange 핸들러가 너무 늦게 실행됨
  useEffect(() => {
    onChange(isOn);
  }, [isOn, onChange])
  ```
### 순서는 다음과 같다.

1. Toggle이 먼저 state를 업데이트

2. React는 Effect를 실행하고 부모 컴포넌트에 전달된 onChange 함수를 호출

3. 부모 컴포넌트는 자신의 state를 업데이트하고 다른 렌더링 패스를 시작

이 과정을 한 번의 패스로 처리하는 것이 좋다.

```
function Toggle({ onChange }) {
  const [isOn, setIsOn] = useState(false);

  function updateToggle(nextIsOn) {
    // ✅ 좋습니다: 업데이트를 유발한 이벤트가 발생한 동안 모든 업데이트를 수행합니다.
    setIsOn(nextIsOn);
    onChange(nextIsOn);
  }

  function handleClick() {
    updateToggle(!isOn);
  }

  function handleDragEnd(e) {
    if (isCloserToRightEdge(e)) {
      updateToggle(true);
    } else {
      updateToggle(false);
    }
  }
 ```

이 접근 방식을 사용하면 Toggle 컴포넌트와 그 부모 컴포넌트 모두 이벤트가 진행되는 동안 state를 업데이트 한다.

 

React는 서로 다른 컴포넌트의 업데이트를 일괄 처리하므로 렌더링 패스는 한 번만 발생한다.

 

### 자식에서 관리하는 state를 완전히 제거하고 부모 컴포넌트로부터 isOn을 수신한다.
```
// ✅ 이것도 좋습니다: 컴포넌트는 부모에 의해 완전히 제어됨.
function Toggle({ isOn, onChange }) {
  function handleClick() {
    onChange(!isOn);
  }

  function handleDragEnd(e) {
    if (isCloserToRightEdge(e)) {
      onChange(true);
    } else {
      onChange(false);
    }
  }

  // ...
}
```
 본적 있죠?

"state를 끌어올리기"는 부모 컴포넌트가 부모 자체의 state를 토글하여 Toggle을 완전히 제어할 수 있게 해준다.

 

즉, 부모 컴포넌트에 더 많은 로직을 포함해야 하지만 전체적으로 신경써야할 state는 줄어든다.

 

### 이때 자식에서 useEffect를 통해 부모에게 데이터를 전달하는 코드는 지양해야한다.
 
```
function Parent() {
  const [data, setData] = useState(null);
  // ...
  return <Child onFetched={setData} />;
}

function Child({ onFetched }) {
  const data = useSomeAPI();
  // 🔴 피하세요: Effect에서 부모에게 데이터 전달하기
  useEffect(() => {
    if (data) {
      onFetched(data);
    }
  }, [onFetched, data]);
  // ...
}
 ```

자식과 부모 모두 동일한 데이터가 필요하므로 부모 컴포넌트가 해당 데이터를 가져와서 자식에게 대신 내려주어야 한다.
```
function Parent() {
  const data = useSomeAPI();
  // ...
  // ✅ 좋습니다: 자식에게 데이터 전달하기
  return <Child data={data} />;
}

function Child({ data }) {
  // ...
}
```