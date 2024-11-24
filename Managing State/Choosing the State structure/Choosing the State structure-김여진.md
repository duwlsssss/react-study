### 1. State 구조화 원칙

오류 없이 state를 업데이트하기 위해 구조화를 해야함

- 연관된 state는 그룹화 고려
두 개의 state 변수가 항상 함께 변경된다면, 단일 state 변수로 통합하기
```jsx
const [x, setX] = useState(0);
const [y, setY] = useState(0);
//대신
const [position, setPosition] = useState({
  x: 0,
  y: 0 
});

// + 필요한 state 수를 모를때 사용시 커스텀 필드를 추가할 수 있어 유용
const [formFields, setFormFields] = useState({
  firstName: '',
  lastName: '',
});

function addField(fieldName) {
  setFormFields(prevFields => ({
    ...prevFields,
    [fieldName]: '', // 새 필드 추가
  }));
}
...
<button onClick={() => addField('email')}>Add Email Field</button>
```

- state 모순 피하기

```jsx
const [text, setText] = useState('');
const [isSending, setIsSending] = useState(false);
const [isSent, setIsSent] = useState(false);
//⚠️ isSending,isSent로 나눠 놓으면 isSending과 isSent가 동시에 true인 불가능한 상황에 처할 위험 있음

// 하나의 state로 통합
const [status, setStatus] = useState('typing');

async function handleSubmit(e) {
  e.preventDefault();
  setStatus('sending');
  await sendMessage(text);
  setStatus('sent');
}
const isSending = status === 'sending';
const isSent = status === 'sent';

if (isSent) {
  return <h1>Thanks for feedback!</h1>
}
```

- state 중복 피하기

동기화를 안 해도 되게 불필요하고 중복된 state를 피해야 함

```jsx
const [items, setItems] = useState(initialItems);
const [selectedItem, setSelectedItem] = useState(
  items[0]
);
// 위 처럼 쓰면 item 수정시 selectedItem도 업데이트 해야함 -> 수동으로 연쇄 업데이트 필요
// items = [{ id: 0, title: 'pretzels'}, ...]
// selectedItem = {id: 0, title: 'pretzels'}
// 대신
const [items, setItems] = useState(initialItems);
const [selectedId, setSelectedId] = useState(0);

const selectedItem = items.find(item =>
  item.id === selectedId
);
// selectedId를 state로 만들고 item 수정시 items 배열에서 selectedId에 해당하는 selectedItem은 렌더링 중에 가져올 수 있음 -> 중복 사라지고 필수 state만 유지됨
// items = [{ id: 0, title: 'pretzels'}, ...]
// selectedId = 0
```
- 특별히 업데이트를 방지하려는 경우를 제외하고는 props를 state에 넣지 말기

props가 업데이트되도 state는 업데이트 안됨

- 깊게 중첩된 state 피하기

중첩된 state를 업데이트하려면 변경된 부분부터 모든 중첨 객체의 복사본을 만들어야 함. 이는 매우 비효율적이고 가독성이 좋지 않은 코드 생성 가능
=> 평탄화해야함 

선택 UI 패턴일 경우 객체 자체가 아니라 ID를 state에 넣어 유지

```jsx
// 객체가 깊게 중첩된 state는 업데이트가 어려움
export const initialTravelPlan = {
  id: 0,
  title: '(Root)',
  childPlaces: [{
    id: 1,
    title: 'Earth',
    childPlaces: [{
      id: 2,
      title: 'Africa',
      childPlaces: [{
        id: 3,
        title: 'Botswana',
        childPlaces: []
      }, {
      ... 

// 각 place가 자식 장소의 배열을 가지는 트리구조 대신
// 각 장소가 자식 장소ID 배열을 가지게 해 장소 매핑

export const initialTravelPlan = {
  0: {
    id: 0,
    title: '(Root)',
    childIds: [1, 42, 46],
  },
  1: {
    id: 1,
    title: 'Earth',
    childIds: [2, 10, 19, 26, 34]
  },
  2: {
    id: 2,
    title: 'Africa',
    childIds: [3, 4, 5, 6 , 7, 8, 9]
  }, 
  3: {
    id: 3,
    title: 'Botswana',
    childIds: []
  },
  ...

const [plan, setPlan] = useState(initialTravelPlan);

// 어떤 장소를 삭제할 때
// 업데이트된 버전의 부모장소는 childIds 배열에서 제거된 ID를 제외해야 함
// 업데이트된 버전의 루트 객체는 부모 장소의 업데이트된 버전을 포함해야 함
function handleComplete(parentId, childId) {
  const parent = plan[parentId];
  const nextParent = {
    ...parent,
    childIds: parent.childIds
      .filter(id => id !== childId)
  };
  setPlan({
    ...plan,
    [parentId]: nextParent
  });
}
```






