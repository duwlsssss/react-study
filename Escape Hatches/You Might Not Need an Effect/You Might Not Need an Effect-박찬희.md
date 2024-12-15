# Effect가 필요하지 않을 수 있습니다.

## 불필요한 Effect를 제거하는 법

Effect가 필요하지 않은 경우는 일반적으로 두 가지가 있습니다.

1. 랜더링을 위해 데이터를 변환할 때 Effect는 필요하지 않습니다. 
   state를 업데이트 할 때 React는 먼저 컴포넌트 함수를 호출해 화면에 표시될 내용을 계산합니다.
   그런 다음 React는 이러한 변경 사항을 DOM에 커밋하여 화면을 업데이트합니다. 그 후 Effect가 실행됩니다.
   만약 Effect도 즉시 state를 업데이트 한다면 전체 프로세스가 처음부터 다시 시작됩니다. 불필요한 랜더링 패스를 피하려면, 컴포넌트의 최상위 레벨에서 모든 데이터를 변환해야합니다.

2. 사용자 이벤트를 처리하는 데 Effect는 필요하지 않습니다.
   제품을 구매할때 POST 요청을 전송하고 알림을 표시하고 싶다고 가정하겠습니다. 구매 버튼 클릭 이벤트 핸들러는 정확히 어떤 일이 일어났는지 알 수 있습니다.
   하지만 Effect의 경우 Effect가 실행될 때까지 사용자가 어떤 버튼을 클릭했는지 알 수 없습니다. 그렇기 때문에 일반적으로 해다오디는 이벤트 핸들러에서 사용자 이벤트를 처리합니다. 

물론 외부 시스템과 동기화하려면 Effect가 반드시 필요합니다. jQuery 위젯이 React state와 동기화 되도록 유지하는 Effect를 작성하는 것 처럼말입니다.
검색 결과를 현재 검색 쿼리와 동기화한다면 Effect로 데이터도 가져올 수 있습니다.
모던 프레임워크는 컴포넌트에 직접 Effect를 작성하는 것보다 더 효율적인 데이터 가져오기 메커니즘을 제공하고 있습니다.

제대로 Effect를 사용하기 위해 몇 가지 일반적이며 구체적인 예를 살펴보겠습니다.

### props 또는 state에 따라 state 업데이트하기

`firstName`과 `lastName`이라는 두 개의 state 변수가 있다고 가정해 봅시다. 두 변수를 연결해서 `fullName`을 계산하고 싶습니다. 또한 두 state가 변경될 때 마다 `fullName`이 업데이트 되기를 바랍니다. 이때 `fullName` state 변수를 추가하고 Effect에서 업데이트 하고 싶을 수 있습니다.
```js
function Form() {
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');

  // 🔴 피하세요: 중복된 state 및 불필요한 Effect
  const [fullName, setFullName] = useState('');
  useEffect(() => {
    setFullName(firstName + ' ' + lastName);
  }, [firstName, lastName]);
  // ...
}
```
이것은 필요 이상으로 복잡합니다. `useState`에서 알아봤던 것처럼 이 예제는 `firstName`과 `lastName`을 통해 `fullName`을 얻어낼 수 있습니다. 그렇다면 계산할 수 있죠. 코드도 빨라지고, 에러가 덜 발생합니다.
하지만 위 예제와 같이 한다면 `fullName`에 대한 오래된 값으로 전체 랜더링을 수행한 이후 업데이트 된 값으로 즉시 다시 랜더링하기 때문에 비효율적입니다.

### 비용이 많이 드는 계산 캐싱하기

다음 컴포넌트는 props로 받은 `todos`를 `filter` prop에 따라 필터링하여 `visibleTodos`를 계산합니다. 결과를 state에 저장하고 Effect에서 업데이트 하고 싶을 수 있죠. 하지만 앞선 예제와 같이 계산할 수 있다면 Effect로 다시 업데이트하고 랜더링하는 것은 좋은 수단이 아닙니다. 그리고 아래와 같이 작성할 수 있습니다.
```jsx
function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');
  // ✅ getFilteredTodos()가 느리지 않다면 괜찮습니다.
  const visibleTodos = getFilteredTodos(todos, filter);
  // ...
}
```
이때 `getFilteredTodos()`가 느리거나 `todos`가 많을 수 있습니다. 이런 경우 `newTodo`와 같이 관련 없는 state 변수가 변경된 경우 `getFilteredTodos()`를 다시 계산하고 싶지 않을 수 있습니다.
이때 useMemo Hook으로 래핑해서 값비싼 계산을 캐시할 수 있습니다.
```jsx
import { useMemo, useState } from 'react';

function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');
  const visibleTodos = useMemo(() => {
    // ✅ todos 또는 filter가 변경되지 않는 한 다시 실행되지 않습니다.
    return getFilteredTodos(todos, filter);
  }, [todos, filter]);
  // ...
}

 // 정리하면 아래와 같습니다.

 function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');
  // ✅ todos나 filter가 변경되지 않는 한 getFilteredTodos()를 다시 실행하지 않습니다.
  const visibleTodos = useMemo(() => getFilteredTodos(todos, filter), [todos, filter]);
  // ...
}
```

이렇게하면 `todos`나 `filter`가 변경되지 않는 한 내부 함수가 다시 실행되지 않기를 원한다고 React에게 알릴 수 있습니다. React는 초기 랜더링 중에 `getFilteredTodos()`의 반환값을 기억하고 다음 랜더링 중에 `todos`나 `filter`가 다른지 확인합니다.
지난번과 동일하다면, `useMemo`는 마지막으로 저장환 결과를 반환하고 다르다면 내부함수를 다시 호출해 그 결과를 저장할 것입니다.

### prop 변경 시 모든 state 초기화

이 `ProfilePage` 컴포넌트는 `userId` prop을 받습니다. 페이지는 댓글 입력을 포함하며 `comment` state 변수를 사용해 해당 값을 보관합니다. 한 프로필에서 다른 프로필로 이동할 때 `comment` state가 재설정되지 않는 문제가 발견되었을 때, 실수로 잘못된 사용자의 프로필 댓글을 게시하기 쉽습니다.
이 문제를 해결하기 위해 `userId`가 변경될 때마다 `comment` state 변수를 비우려고 합니다.
```jsx
export default function ProfilePage({ userId }) {
  const [comment, setComment] = useState('');

  // 🔴 피하세요: Effect에서 prop 변경 시 state 초기화
  useEffect(() => {
    setComment('');
  }, [userId]);
  // ...
}
```
이것은 `ProfilePage`와 그 자식이 오래된 값으로 랜더링 한 다음 한번 더 랜더링 하기 때문에 비효율적입니다.
또한 `ProfilePage` 내부에 어떤 state가 있는 모든 컴포넌트에서 이 작업을 수행해야 하므로 복잡합니다.

대신 명시적인 key를 전달하여 각 사용자의 프로필이 개념적으로 다른 프로필임을 React에게 알릴 수 있습니다.
컴포넌트를 둘로 분할하고 외부 컴포넌트에서 내부 컴포넌트로 `key`어트리뷰트를 전달하는것입니다.

```jsx
export default function ProfilePage({ userId }) {
  return (
    <Profile
      userId={userId}
      key={userId}
    />
  );
}

function Profile({ userId }) {
  // ✅ 이 state 및 아래의 다른 state는 key 변경 시 자동으로 재설정됩니다.
  const [comment, setComment] = useState('');
  // ...
}
```
`Profile` 컴포넌트에 `userId`를 `key`로 전달하면 React가 `userId`가 다른 두 컴포넌트를 state를 공유해서는 안되는 개별의 컴포넌트로 취급하도록 요청하는 것입니다.

### prop이 변경될 때 일부 state 조정하기

prop이 변경될 때 전체가 아닌 일부 state만 재설정하거나 조정하고 싶을 때가 있습니다.
다음 `List` 컴포넌트는 `items` 목록을 prop으로 받고 `selection` state 변수에 선택된 item을 유지합니다. `items` prop이 다른 배열을 받을 때 마다 `selection`을 `null`로 재설정하고 싶습니다.
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
이렇게하면 `items`가 변경될 때 마다 `List`와 그 자식 컴포넌트들은 처음에는 이전 `selection`값으로 랜더링되고 이후 React가 DOM을 업데이트하고 Effect를 실행합니다.
Effect가 실행되면 `setSelection(null)`호출은 `List`와 그 자식 컴포넌트들을 다시 랜더링하여 한 번 더 이 전체 프로세스가 다시 시작하게 됩니다.

이런 경우에도 Effect는 좋지 않은 선택입니다. 랜더링 중 state를 직접 조정하는 것이 좋습니다.

```js
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selection, setSelection] = useState(null);

  // 더 좋습니다: 렌더링 중 state 조정
  const [prevItems, setPrevItems] = useState(items);
  if (items !== prevItems) {
    setPrevItems(items);
    setSelection(null);
  }
  // ...
}
```
위 예시의 경우 랜더링 도중 `setSelection`이 호출됩니다. React는 아직 `return`문으로 종료된 후 즉시 List를 다시 랜더링 합니다. 이전과 같이 오래된 selection 값의 랜더링은 건너 뛸 수 있지만 랜더링 중이던 `List` 컴포넌트를 다시 처음부터 랜더링해야됩니다.

이러한 매우 느린 연속적 재시도를 피하기 위해 React는 랜더링 중에 동일한 컴포넌트의 state만 업데이트할 수 있도록 해야합니다. 랜더링 도중 다른 컴포넌트의 state를 업데이트하면 에러가 발생합니다. 이를 방지하기 위해서는 `items !== prevItems`와 같은 조건이 필요합니다. 이런식으로 state를 조정할 수 있지만 컴포넌트를 순수하게 유지하기 위해서는 DOM 변경이나 타임아웃 설정과 같은 사이드 이펙트들은 이벤트 핸들러나 Effect에 남겨둬야합니다.

이 패턴이 Effect보다는 효율적이지만 대부분의 컴포넌트에서 이 패턴은 필요로하지 않습니다. 결국 props나 다른 state에 따라 state를 조정하면 데이터 흐름을 이해하고 디버깅하기 어려워집니다.
대신 key를 사용하여 모든 state를 초기화하거나 랜더링 중 모든 state를 계산할 수 있는지 확인해야합니다.

```jsx
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selectedId, setSelectedId] = useState(null);
  // ✅ 최고예요: 렌더링 중에 모든 것을 계산
  const selection = items.find(item => item.id === selectedId) ?? null;
  // ...
}
```
위 예제의 경우 state를 더이상 조정할 필요가 없습니다. 선택한 ID를 가진 item이 목록에 있으면 선택된 state로 유지됩니다. 그렇지 않은 경우 일치하는 item을 찾을 수 없으므로 랜더링 중에 계산된 `selection`은 `null`이 됩니다.

### 이벤트 핸들러 간 로직 공유 

제품을 구매할 수 있는 두 개의 버튼이 있는 제품 페이지가 있다고 가정해보겠습니다. 사용자가 제품을 넣을 때마다 알림을 표시하고 싶습니다. 두 버튼을 클릭 핸들러에서 모두 `showNotification()`을 호출하는 것은 반복적으로 느껴지므로 이 로직을 Effect에 배치하고 싶을 수 있습니다.
```jsx
function ProductPage({ product, addToCart }) {
  // 🔴 피하세요: Effect 내부의 이벤트별 로직
  useEffect(() => {
    if (product.isInCart) {
      showNotification(`Added ${product.name} to the shopping cart!`);
    }
  }, [product]);

  function handleBuyClick() {
    addToCart(product);
  }

  function handleCheckoutClick() {
    addToCart(product);
    navigateTo('/checkout');
  }
  // ...
}
```
하지만 이 Effect는 불필요합니다. 또한 버그 유발 가능성이 높습니다.
페이지가 리로드 될 때마다 앱이 장바구니를 기억한다고 가정할 떄, 카트에 제품을 한 번 추가하고 새로고침하면 알림이 다시 표시됩니다. 매번 새로 고침할때마다 표시됩니다. 이는 페이지 로드시 `product.isInCart`가 이미 `true`이기 때문에 Effect는 `showNotification()`을 호출합니다.

**컴포넌트가 사용자에게 표시되었기 떄문에 실행되어야하는 코드에만 Effect를 사용해야합니다.**
이 예시는 페이지가 표시되었기 때문이 아닌 사용자가 버튼을 눌렀기 때문에 알림이 표시되어야 합니다. 따라서 Effect는 필요 없습니다.
다음과 같이 수정해봅시다.
```jsx
function ProductPage({ product, addToCart }) {
  // ✅ 좋습니다: 이벤트 핸들러에서 이벤트별 로직이 호출됩니다.
  function buyProduct() {
    addToCart(product);
    showNotification(`Added ${product.name} to the shopping cart!`);
  }

  function handleBuyClick() {
    buyProduct();
  }

  function handleCheckoutClick() {
    buyProduct();
    navigateTo('/checkout');
  }
  // ...
}
```

### POST 요청 보내기

이 `Form` 컴포넌트는 두 가지 종류의 POST 요청을 전송합니다. 마운트 될 때 analytics 이벤트를 보냅니다. 폼을 작성하고 Submit 버튼을 클릭하면 `/api/register` 엔드포인트로 POST 요청을 보냅니다.

```jsx
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

  function handleSubmit(e) {
    e.preventDefault();
    setJsonToSubmit({ firstName, lastName });
  }
  // ...
}
```
analytics POST 요청은 폼이 표시되어 analytics 이벤트를 전송해야하기 때문이기 때문에 Effect에 남아있어야합니다.
하지만 `/api/register` POST 요청은 표시되는 폼으로 인해 발생하는 것이 아닙니다. 따라서 특정 상호작요에서만 발생하는 것이고, 두 번째 Effect를 삭제하고 해당 POST요청을 이벤트 핸들러로 이동시켜야합니다.

```jsx
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
로직을 이벤트 핸들러에 넣을지 Effect에 넣을지 선택할 때 사용자 관점에서 어떤 종류의 로직인지 고려해봐야합니다.
특정 상호작용으로 인해 발생하는 것이라면 이벤트 핸들러에 사용자 화면에서 컴포넌트를 보는 것이 원인이라면 Effect에 넣어야합니다.

### 연쇄 계산

다른 state에 따라 각각 state를 조정하는 Effect를 체이닝하는 예제가 있습니다.
```jsx
function Game() {
  const [card, setCard] = useState(null);
  const [goldCardCount, setGoldCardCount] = useState(0);
  const [round, setRound] = useState(1);
  const [isGameOver, setIsGameOver] = useState(false);

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

  useEffect(() => {
    if (round > 5) {
      setIsGameOver(true);
    }
  }, [round]);

  useEffect(() => {
    alert('Good game!');
  }, [isGameOver]);

  function handlePlaceCard(nextCard) {
    if (isGameOver) {
      throw Error('Game already ended.');
    } else {
      setCard(nextCard);
    }
  }
```
이 코드에는 두가지 문제가 있습니다.
1. 매우 비효율적입니다. 컴포넌트는 체인의 각 `set` 호출 사이에 다시 랜더링해야합니다. 최악의 경우 아래 트리의 불필요한 리랜더링이 세 번 발생합니다.

2. 속도가 느리지 않더라도 코드가 발전함에 따라 작성한 체인이 새로운 요구에 맞지 않는 상황이 발생할 수 있다는 점입니다. 게임 이동의 기록을 단계별로 살펴볼 수 있는 방법을 추가한다고 가정했을때, 각 state 변수를 과거의 값으로 업데이트하여 이를 수행할 수 잇습니다.
하지만 `card` state를 과거의 값으로 설정하면 Effect 체인이 다시 트리거되고 표시되는 데이터가 변경됩니다. 이러한 코드는 융통성이 없고 취약한 경우가 많습니다.

이러한 경우 랜더링 중에 가능한 것을 계산하고 이벤트 핸들러에서 state를 조정하는 것이 좋습니다.

```jsx

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
        setRound(round + 1);
        if (round === 5) {
          alert('Good game!');
        }
      }
    }
  }
```
훨씬 효율적입니다. 또한 게임 기록을 볼 수 있는 방법을 구현하면 이제 다른 모든 값을 조정하는 Effect 체인을 트리거하지 않고 각 state 변수를 과거의 행동으로 설정할 수 있습니다.
여러 이벤트 핸들러 간에 로직을 재사용해야하는 경우 함수를 추출하여 해당 핸들러에서 호출할 수 있습니다.

이벤트 핸들러 내부에서 state는 스냅샷처럼 동작한다는 점을 기억하세요. 
또한 이벤트 핸들러에서 직접 다음 state를 계산할 수 없는 경우도 있습니다. 여러 개의 드롭 다운이 있는 폼에서 다음 드롭 다운의 옵션이 이전 드롭 다운의 선택된 값에 따라 달라진다고 가정해보겠습니다. 이 경우 네트워크와 동기화하기 때문에 Effect 체인이 적절합니다.

### 애플리케이션 초기화 
일부 로직은 앱이 로드될 때 한 번만 실행되어야 합니다.
그것을 최상위 컴포넌트의 Effect에 배치하고 싶을 수도 있습니다.
```jsx
function App() {
  // 🔴 피하세요: 한 번만 실행되어야 하는 로직이 포함된 Effect
  useEffect(() => {
    loadDataFromLocalStorage();
    checkAuthToken();
  }, []);
  // ...
}
```
하지만 이 함수는 개발 중에 두 번 실행됩니다. React의 strick mode 때문이죠. 그에 비해 함수가 두 번 호출되도록 설계되지 않았기 때문에 인증 토큰이 무효화되는 등의 문제가 발생할 수 있습니다.

프로덕션 환경에서 실제로 다시 마운트되지 않을 수도 있지만 모든 컴포넌트에서 동일한 제약 조건을 따르면 코드를 이동하고 재사용하기가 더 쉬워집니다. 일부 로직이 컴포넌트 마운트 당 한 번이 아니라 앱 로드당 한 번 실행되어야 하는 경우 최상위 변수를 추가하여 이미 실행되었는지를 추적하세요.

```js
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

모듈 초기화 중이나 앱이 랜더링 되기 전에 실행하기를 원한다면 다음과 같이 작성할 수 있습니다.
```jsx
if (typeof window !== 'undefined') { // 브라우저에서 실행 중인지 확인합니다.
   // ✅ 앱 로드당 한 번만 실행
  checkAuthToken();
  loadDataFromLocalStorage();
}

function App() {
  // ...
}
```

컴포넌트를 import할 때 최상위 레벨의 코드는 랜더링 되지 않더라도 한 번 실행됩니다. 임의의 컴포넌트를 import 할때 속도 저하나 예상치 못한 동작을 방지하려면 이 패턴은 과도하게 사용하지 말아야합니다.
app 전체 초기화 로직은 `App.js`와 같은 루트 컴포넌트의 모듈이나 애플리케이션의 엔트리 포인트에 두세요.

### state 변경을 부모 컴포넌트에게 알리기

`true` 또는 `false`가 될 수 있는 내부 `isOn` state를 가진 `Toggle` 컴포넌트를 작성하고 있다고 가정해봅시다. 클릭 또는 드래그를 통해 토글하는 방법에는 몇 가지가 있습니다. `Toggle` 내부 state가 변경될 때마다 부모 컴포넌트에 알리고 싶을 때 `onChange` 이벤트를 노출하고 Effect에서 호출합니다.

```jsx
function Toggle({ onChange }) {
  const [isOn, setIsOn] = useState(false);

  // 🔴 피하세요: onChange 핸들러가 너무 늦게 실행됨
  useEffect(() => {
    onChange(isOn);
  }, [isOn, onChange])

  function handleClick() {
    setIsOn(!isOn);
  }

  function handleDragEnd(e) {
    if (isCloserToRightEdge(e)) {
      setIsOn(true);
    } else {
      setIsOn(false);
    }
  }

  // ...
}
```

`Toggle`이 먼저 state를 업데이트하고 React가 화면을 업데이트합니다. 그 후 React는 Effect를 실행하고 부모 컴포넌트에서 전달된 `onChange`함수를 호출합니다. 이제 부모 컴포넌트는 자신의 state를 업데이트하고 다른 랜더링 패스를 시작합니다. 이 모든 것을 한 번의 패스로 처리하는 것이 좋습니다.

```jsx
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

  // ...
}
```

이 접근 방식을 사용하면 `Toggle` 컴포넌트와 그 부모 컴포넌트 모두 이벤트가 진행되는 동안 state를 업데이트합니다. React는 서로 다른 컴포넌트의 업데이트를 일괄 처리하므로 랜더링 패스는 한 번만 발생합니다.

state를 완전히 제거하고 대신 부모컴포넌트로부터 `isOn`을 수신할 수도 있습니다.
```jsx
// ✅ 이것도 좋습니다: 컴포넌트는 부모에 의해 완전히 제어됩니다.
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

state 끌어올리기는 부모 컴포넌트가 부모 자체의 state를 토글하여 `Toggle`을 완전히 제어할 수 있게 해줍니다. 즉, 부모 컴포넌트에 더 많은 로직을 포함해야 하지만 전체적인 state는 줄어듭니다. 두 서로 다른 state 변수를 동기화하려고 할 때마다 state 끌어올리기를 사용해보세요.

### 부모에게 데이터 전달하기

이 `Child` 컴포넌트는 일부 데이터를 가져온 다음 Effect에서 `Parent` 컴포넌트에 전달합니다.
```jsx
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

React에서 데이터는 부모 컴포넌트에서 자식 컴포넌트로 흐릅니다. 화면에 잘못된 것이 보이면 컴포넌트 체인을 따라 올라가 어떤 컴포넌트가 잘못된 prop을 전달하거나 state를 가지고 있는지 찾아내면 정보의 출처를 추적할 수 있습니다. 자식 컴포넌트가 Effect에서 부모 컴포넌트의 state를 업데이트하면 데이터 흐름을 추적하기 매우 어려워집니다. 자식과 부모 모두 동일한 데이터가 필요하므로 부모 컴포넌트가 해당 데이터를 가져와서 자식에게 대신 내려주도록 하세요.

```jsx
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
### 외부 저장소 구독하기
때로는 컴포넌트가 React state 외부의 일부 데이터를 구독해야 할 수도 있습니다. 이 데이터는 서드파티 라이브러리 또는 내장 브라우저 API에서 가져올 수 있습니다. 이 데이터는 React가 모르는 사이에 변경될 수 있으므로 컴포넌트를 수동으로 구독해야합니다. 이 작업은 종종 Effect를 통해 수행됩니다.
```jsx
function useOnlineStatus() {
  // 이상적이지 않습니다: Effect에서 저장소를 수동으로 구독
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    function updateState() {
      setIsOnline(navigator.onLine);
    }

    updateState();

    window.addEventListener('online', updateState);
    window.addEventListener('offline', updateState);
    return () => {
      window.removeEventListener('online', updateState);
      window.removeEventListener('offline', updateState);
    };
  }, []);
  return isOnline;
}

function ChatIndicator() {
  const isOnline = useOnlineStatus();
  // ...
}
```
React에는 외부 저장소를 구독하기 위해 제작된 Hook이 있습니다. Effect 대신 `useSyncExternalStore`를 호출합니다.
```js
function subscribe(callback) {
  window.addEventListener('online', callback);
  window.addEventListener('offline', callback);
  return () => {
    window.removeEventListener('online', callback);
    window.removeEventListener('offline', callback);
  };
}

function useOnlineStatus() {
  // ✅ 좋습니다: 내장 Hook으로 외부 스토어 구독하기
  return useSyncExternalStore(
    subscribe, // 동일한 함수를 전달하는 한 React는 다시 구독하지 않습니다.
    () => navigator.onLine, // 클라이언트에서 값을 얻는 방법
    () => true // 서버에서 값을 얻는 방법
  );
}

function ChatIndicator() {
  const isOnline = useOnlineStatus();
  // ...
}
```
이 접근 방식은 변경 가능한 데이터를 Effect를 사용해 React state에 수동으로 동기화하는 것보다 에러가 덜 발생합니다.
일반적으로 `useOnlineStatus()`와 같은 사용자 정의 Hook을 작성하여 개별 컴포넌트에서 이 코드를 반복할 필요가 없도록 합시다.

### 데이터 가져오기

많은 앱이 데이터를 가져오기 시작하기 위해 Effect를 사용합니다. 이와 같은 데이터를 가져오는 Effect를 작성하는 것은 일반적입니다.
```jsx
function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const [page, setPage] = useState(1);

  useEffect(() => {
    // 🔴 피하세요: 정리 로직 없이 가져오기
    fetchResults(query, page).then(json => {
      setResults(json);
    });
  }, [query, page]);

  function handleNextPageClick() {
    setPage(page + 1);
  }
  // ...
}
```
데이터 가져오기를 이벤트 핸들러로 옮길 필요는 없습니다.
이벤트 핸들러에 로직을 넣어야했던 앞선 예시와 모순되는 것처럼 보일 수 있습니다. 하지만 데이터 가져오기를 해야하는 주된 이유가 입력 이벤트가 아니라는 점을 고려해야합니다. 검색입력은 URL에서 미리 채워지는 경우가 많으며 사용자는 입력을 건드리지 않고 뒤로 앞으로 탐색할 수도 있습니다.

`page`와 `query`의 출처가 어디인지는 중요치 않습니다. 이 컴포넌트가 표시되는 동안에는 현재 `page` 및 `query`에 대한 네트워크 데이터와 `results`를 동기화하고 싶을 것입니다.

하지만 위 코드에는 버그가 있습니다. 빠르게 검색어를 입력하는 경우 별도의 데이터를 가져오기가 시작되지만 어떤 순서로 응답이 도착할지 보장할 수 없습니다.
`setResults()`를 마지막으로 호출하므로 잘못된 검색 결과가 표시될 수 있습니다. 
이를 경쟁 조건이라 하는데 서로 다른 두 요청이 서로 경쟁하여 예상과 다른 순서로 도착하는 것을 말합니다.

경쟁 조건을 수정하려면 오래된 응답을 무시하는 정리 함수가 추가되어야 합니다.

```jsx
function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const [page, setPage] = useState(1);
  useEffect(() => {
    let ignore = false;
    fetchResults(query, page).then(json => {
      if (!ignore) {
        setResults(json);
      }
    });
    return () => {
      ignore = true;
    };
  }, [query, page]);

  function handleNextPageClick() {
    setPage(page + 1);
  }
  // ...
}
```
이렇게하면 Effect가 데이터를 가져올 때 마지막으로 요청된 응답을 제외한 모든 응답이 무시됩니다.

데이터 가져오기를 구현할 때 경쟁 조건을 처리하는 것만이 어려운 것은 아닙니다.
응답 캐싱, 서버에서 데이터를 가져오는 방법, 네트워크 워터폴을 피하는 방법도 고려해야합니다.

이러한 문제는 React뿐 아닌 모든 UI 라이브러리에 적용됩니다. 문제를 해결하는 것은 간단하지 않기 때문에 모던 프레임워크는 Effect에서 데이터를 가져온는 것보다 더 효율적인 내장 데이터 가져오기 메커니즘을 제공합니다.

프레임워크를 사용하지 않고 Effect에서 데이터를 보다 인체공학적으로 가져오고 싶다면 아래 예시를 참고해보세요.
```jsx
function SearchResults({ query }) {
  const [page, setPage] = useState(1);
  const params = new URLSearchParams({ query, page });
  const results = useData(`/api/search?${params}`); //

  function handleNextPageClick() {
    setPage(page + 1);
  }
  // ...
}

function useData(url) {
  const [data, setData] = useState(null);
  useEffect(() => {
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
  }, [url]);
  return data;
}
```
이 방법만으로는 프레임워크에 내장된 데이터 가져오기 메커니즘을 사용하는 것 만큼 효율적이지는 않지만, 데이터 가져오기 로직을 사용자 정의 Hook으로 옮기면 나중에 효율적인 데이터 가져오기 전략을 취하기 더 쉬워집니다.
일반적으로 Effect를 작성해야 할 때마다 위의 `useData`와 같이 선언적이고 목적에 맞게 구축된 API를 사용하여 일부 기능을 커스텀 Hook으로 추출할 수 있는 경우를 주시하세요 컴포넌트에서 원시 `uesEffect` 호출이 적을수록 애플리케이션 유지관리가 쉬워집니다.