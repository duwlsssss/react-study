## useEffect가 필요한 경우와 필요 없는 경우
#### 필요한 경우
- eact 외부 시스템과 동기화: 외부 API, 브라우저 DOM, 네트워크 등.
- 컴포넌트가 화면에 표시되거나 상태가 변할 때 실행:
- 브라우저 이벤트 구독 (예: scroll, resize).
- 데이터 가져오기.
- DOM 조작.
#### 필요 없는 경우
1. 렌더링 중 계산 가능:
- 계산이 React의 렌더링 과정에서 수행될 수 있다면, useEffect 대신 바로 계산.
- 예: props나 state를 기반으로 값을 계산.
2. 사용자 이벤트 처리:
- 특정 사용자 동작으로만 실행되어야 하는 로직은 이벤트 핸들러로 처리.
- 예: 버튼 클릭 시 네트워크 요청 전송.


## 불필요한 useEffect를 제거하는 방법
#### 1. 렌더링 중 데이터 변환
- 예제: firstName과 lastName을 결합하여 fullName을 생성.

``` javascript

// 🔴 비효율적: Effect를 사용한 중복 계산
function Form() {
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');
  const [fullName, setFullName] = useState('');

  useEffect(() => {
    setFullName(`${firstName} ${lastName}`);
  }, [firstName, lastName]);
}
```
수정: 렌더링 중 계산.

``` javascript

// ✅ 렌더링 중 계산
function Form() {
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');
  const fullName = `${firstName} ${lastName}`;
}
```
#### 2. 값비싼 계산 캐싱
예제: todos를 필터링하는 함수가 느린 경우.

``` javascript

// 🔴 비효율적: Effect와 state 사용
function TodoList({ todos, filter }) {
  const [visibleTodos, setVisibleTodos] = useState([]);
  
  useEffect(() => {
    setVisibleTodos(getFilteredTodos(todos, filter));
  }, [todos, filter]);
}
```
수정: useMemo로 캐싱

``` javascript

// ✅ useMemo로 값비싼 계산 캐싱
function TodoList({ todos, filter }) {
  const visibleTodos = useMemo(() => getFilteredTodos(todos, filter), [todos, filter]);
}
```
#### 3. Prop 변경에 따른 상태 초기화
예제: userId가 바뀔 때마다 댓글 상태를 초기화.

``` javascript

// 🔴 비효율적: Effect로 상태 초기화
function ProfilePage({ userId }) {
  const [comment, setComment] = useState('');

  useEffect(() => {
    setComment('');
  }, [userId]);
}
```
수정: 컴포넌트 분리와 key 속성 사용.

``` javascript

// ✅ key 속성으로 컴포넌트 상태 초기화
function ProfilePage({ userId }) {
  return <Profile key={userId} userId={userId} />;
}

function Profile() {
  const [comment, setComment] = useState('');
}
```
#### 4. 이벤트 핸들러로 로직 이동
예제: 사용자가 버튼 클릭 시 API 호출.

``` javascript

// 🔴 비효율적: Effect로 이벤트 처리
function Form() {
  const [data, setData] = useState(null);

  useEffect(() => {
    if (data) {
      post('/api/register', data);
    }
  }, [data]);

  function handleSubmit(e) {
    e.preventDefault();
    setData({ name: 'Taylor' });
  }
}
```
수정: 이벤트 핸들러에서 API 호출.

``` javascript

// ✅ 이벤트 핸들러로 로직 이동
function Form() {
  function handleSubmit(e) {
    e.preventDefault();
    post('/api/register', { name: 'Taylor' });
  }
}
```
## Effect를 반드시 사용해야 하는 경우
1. 외부 시스템과 동기화
예제: DOM 이벤트 구독 및 정리.

``` javascript

useEffect(() => {
  function handleScroll() {
    console.log('Scrolling...');
  }

  window.addEventListener('scroll', handleScroll);
  return () => window.removeEventListener('scroll', handleScroll);
}, []);
```
2. 데이터 가져오기
예제: 검색 결과 가져오기.

``` javascript

useEffect(() => {
  let ignore = false;

  fetch(`/api/search?q=query`)
    .then(response => response.json())
    .then(data => {
      if (!ignore) {
        setResults(data);
      }
    });

  return () => {
    ignore = true;
  };
}, [query]);
```

## 불필요한 Effect의 문제점
1. 성능 저하: 불필요한 렌더링 및 계산.
2. 코드 복잡성 증가: 중복 코드로 유지보수 어려움.
3. 버그 유발 가능성:상태가 엇갈려 동기화 문제 발생.




