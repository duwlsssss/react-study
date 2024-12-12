외부 시스템이 관여하지 않는 경우(예를 들어 일부 props 또는 state가 변경될 때 컴포넌트의 state를 업데이트 하려는 경우) Effect가 필요하지 않다 useState가 있으니까!

 

불필요한 Effect를 제거하면 코드를 더 쉽게 따라갈 수 있고, 실행 속도가 빨라지며 에러 발생 가능성이 줄어든다

 

### 학습 내용
- 컴포넌트에서 불필요한 Effect를 제거하는 이유와 방법
- Effect 없이 값비싼 계산을 캐시하는 방법
- Effect 없이 컴포넌트 state를 초기화하고 조정하는 방법
- 이벤트 핸들러 간에 로직을 공유하는 방법
- 이벤트 핸들러로 이동해야 하는 로직
- 부모 컴포넌트에 변경 사항을 알리는 방법
 

### 불 필요한 Effect를 제거하는 방법
Effect가 필요하지 않은 두 가지 일반적인 경우가 있다.

- 렌더링을 위해 데이터를 변환하는데 Effect가 필요하지 않다.
    - 불필요한 렌더링 패스를 피하려면, 컴포넌트의 최상위 레벨에서 모든 데이터를 변환하자
    - 그러면 props나 state가 변경될 때마다 해당 코드가 자동으로 다시 실행된다.
- 사용자 이벤트를 처리하는데 Effect가 필요하지 않다.
    - 예를 들어, 구매 버튼 클릭 이벤트 핸들러 안에서는 어떤 일이 일어났는지 알 수 있지만
    - Effect가 실행될 때까지 사용자가 무엇을 했는지는 알 수 없다.
모던 프레임워크는 컴포넌트에 직접 Effect를 작성하는 것보다 더 효율적인 내장 데이터 가져오기 메커니즘을 제공한다

### props 또는 state에 따라 state 업데이트 하기
firstName과 lastName이라는 두 개의 state 값이 존재한다고 가정하여 , 두 변수를 연결해 fullName을 계산하고 싶다.

또한 firstName이나 lastName이 변경될 때 마다 fullName이 업데이트 되길 기대한다.
```
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
위 코드는 fullName에 대한 오래된 값으로 전체 렌더링 패스를 수행한 다음, 업데이트된 값으로 즉시 다시 렌더링하기 때문에 비 효율적이다.

### state 변수와 Effect를 제거
```
function Form() {
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');
  // ✅ 좋습니다: 렌더링 중에 계산됨
  const fullName = firstName + ' ' + lastName;
  // ...
}
 ```

기존 props나 state에서 계산할 수 있는 것이 있으면, 그것을 state에 넣지 말고 렌더링 중에 계산하게 하자

이렇게 하면

더 빨라지고(추가적인 "연속적인" 업데이트를 피할 수 있다)

더 간단해지고(일부 코드를 state, Effect를 제거 )

에러가 덜 발생(서로 다른 state 변수가 서로 동기화되지 않아 발생하는 버그를 피할 수 있다.)

 

### 비용이 많이 드는 계산 캐싱하기
이 컴포넌트는 props로 받은 todos를 filter prop에 따라 필터링하여 visibleTodos를 계산한다.
```
function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');
  // ✅ getFilteredTodos()가 느리지 않다면 괜찮습니다.
  const visibleTodos = getFilteredTodos(todos, filter);
  // ...
}
 ```

만약 getFilteredTodos()가 느리거나 todos가 많다면..?

이 경우 newTodo와 같이 관련 없는 state 변수가 변경된 경우 getFilteredTOdos()를 다시 계산하지 않고 싶을 수 있다.

 

이때 useMemo Hook으로 래핑해서 값비싼 계산 캐시(또는 메모이제이션) 할 수 있다.
```
import { useMoemo, useState } from 'react'

function TodoList({ todos, filter }) {
	const [newTodo, [setNewTodo] = useState('')
    const visibleTodos = useMomo(() => {
    // ✅ todos 또는 filter가 변경되지 않는 한 다시 실행되지 않습니다.
    return getFilteredTodos(todos, filter);
   }, [todos, filter]);
   ...
```
이렇게 작성하면 todos나 filter가 변경되지 않는 한 내부 하무가 다시 실행되지 않기를 원한다는 걸 React에게 알린다.

useMemo로 감싸는 함수는 렌더링 중에 실행되므로, 순수한 계산에만 작동한다.

 

### 어떻게 계산이 비싼지 알 수 있을까?
```
console.time('filter array');
const visibleTodos = getFilteredTodos(todos, filter);
console.timeEnd('filter array');
```
time을 이용하여 총 로깅시간을 계산한다.

 

### Prop 변경 시 모든 state 초기화
ProfilePage 컴포넌트는 userId prop을 받는다.

페이지는 댓글 입력을 포함하며 comment state 변수를 사용해 해당 값을 보관한다.
```
export default function ProfilePage({ userId }) {
  const [comment, setComment] = useState('');

  // 🔴 피하세요: Effect에서 prop 변경 시 state 초기화
  useEffect(() => {
    setComment('');
  }, [userId]);
  // ...
}
```

이는 비효율적인데 ProfilePage와 그 자식이 오래된 값으로 처음 렌더링 한 다음 다시 렌더링하기 때문이다.

또한 ProfilePage 내부에 어떤 state가 있는 모든 컴포넌트에서 이 작업을 수행해야 한다.

예를 들어 댓글 UI가 중첩된 경우 중첩된 댓글 state도 비워야 한다.

 

### 대신 명시적인 key를 전달하여 각 사용자의 프로필이 개념적으로 다른 프로필임을 React에 알릴 수 있따.
```
export default function ProfilePage({ useId }){
	return(
    	<Profile
        	userId={userId}
            key={userId}
        />
    );
 }
 
 function Profile({ userId }){
 	// ✅ 이 state 및 아래의 다른 state는 key 변경 시 자동으로 재설정됩니다.
    const [comment, setComment] = useState('')
 
```
Profile 컴포넌트에 userId를 key로 전달하면 React가 userId가 다른 두 개의 Profile 컴포넌트를 state를 공유해서는 안되는 두 개의 다른 컴포넌트를 취급하도록 요청하는 것이다.

userId로 설정한 key가 변경될 때마다 React는 DOM을 다시 생성하고 Profile 컴포넌트와 그 모든 자식의 state를 재설정한다.

따라서 comment 필드가 자동으로 비워질 것이다(모든 자식까지니까)


### Prop이 변경될 때 일부 state 조정하기
아래의 List 컴포넌트는 items 목록을 prop으로 받고 selection state 변수에 선택된  item을 유지한다.

items rpop이 다른 배열을 받을 때마다 selection을 null로 재설정하고 싶다.
```
function List({ items }){
	const [isReverse, setIsReverse] = useState(false);
    const [selection, setSelection] = useState(null);
    
    // 🔴 피하세요: Effect에서 prop 변경 시 state 조정하기
    useEffect(() => {
    	setSelection(null);
    }, [items]);
 ```
items가 변경될 때마다 List와 그 자식 컴포넌트들은 처음에는 오래된 selection값으로 렌더링 된다.

 

### 그런다음 React는 DOM을 업데이트하고 Effect를 실행한다.

마지막으로 setSelection(null) 호출은 List와 그 자식 컴포넌트를 다시 렌더링하여 이 전체 프로세스를 다시 시작하게 된다.
```
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

React는 return 문으로 종료된 후 즉시 List를 다시 렌더링한다.

REact는 아직 List 자식을 렌더링하거나 DOM을 업데이트하지 않았기 때문에 오래된 selection 값의 렌더링을 건너뛸 수 있다.

렌더링 도중 컴포넌트를 업데이트하면 React는 반환된 JSX를 버리고 즉시 렌더링을 다시 시도한다.

 

### 강추 ! key를 사용하여 모든 state를 초기화하거나 렌더링 중에 모든 state를 계산 
```
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selectedId, setSelectedId] = useState(null);
  // ✅ 최고예요: 렌더링 중에 모든 것을 계산
  const selection = items.find(item => item.id === selectedId) ?? null;
  // ...
}
``` 
### 왜? -> state를 "조정"할 필요가 없기에

 

### 이벤트 핸들러 간 로직 공유
어떤 코드가 Effect에 있어야 하는지 이벤트 핸들러에 있어야 하는지 확실하지 않은 경우 이 코드가 실행되어야 하는 이유를 자문해봐라.

컴포넌트가 사용자에게 표시되었기 때문에 실행되어야 하는 코드에만 Effect를 사용해야한다.
```
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

### 결론은.. Effect를 최대한 사용하지말자


