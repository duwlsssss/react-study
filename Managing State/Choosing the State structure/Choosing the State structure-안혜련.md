# Choosing the State Structure

## State 구조화 원칙

### (1) 연관된 state 병합

```jsx
// 나쁜 예
const [x, setX] = useState(0);
const [y, setY] = useState(0);

// 좋은 예
const [position, setPosition] = useState({ x: 0, y: 0 });
```

두 값이 항상 동시에 업데이트되므로 병합하여 관리.

### (2) 모순된 state를 하나의 변수로 통합

```jsx
// 나쁜 예
const [isSending, setIsSending] = useState(false);
const [isSent, setIsSent] = useState(false);

// 좋은 예
const [status, setStatus] = useState("typing"); // 'typing', 'sending', 'sent'
```

isSending과 isSent는 동시에 true가 되어서는 안되기 때문에, 이 두 변수를 'typing'(초깃값), 'sending', 'sent' 세 가지 유효한 상태 중 하나를 가질 수 있는 status state 변수로 대체하는 것이 좋다!

상태를 통합하여 잘못된 조합(예: isSending과 isSent 둘 다 true)을 방지.

### (3) 불필요한 state 제거

- 기존 state나 props로 계산 가능한 값은 state에 포함하지 않음.
- state는 필요한 최소 데이터만 포함.

```jsx
// 불필요한 state 포함
const [firstName, setFirstName] = useState("");
const [lastName, setLastName] = useState("");
const [fullName, setFullName] = useState("");

// 개선된 방식
const fullName = `${firstName} ${lastName}`; // 렌더링 중 계산
```

fullName은 렌더링 중에 계산하면 충분. 불필요한 state를 줄여 유지보수를 간소화.

### (4) 중복 state 피하기

items 배열과 선택된 항목(selectedItem)을 동시에 state로 관리했다.

```jsx
const [items, setItems] = useState(initialItems); //전체 항목 목록
const [selectedItem, setSelectedItem] = useState(items[0]); ///현재 선택된 항목
```

#### 왜 문제가 될까??

1. 중복 데이터:

- items 배열과 selectedItem은 같은 정보를 중복해서 저장합니다.
- selectedItem은 결국 items 배열의 하나의 객체를 참조하고 있습니다.

2. 동기화 문제:

- 사용자가 항목을 수정하면, items는 업데이트되지만 selectedItem은 업데이트되지 않아, 상태 불일치가 발생합니다.
- 즉, 화면에 업데이트되지 않는 문제!

#### 해결 방법: 중복 제거

selectedItem을 state에서 제거하고, 대신 **선택된 항목의 ID(selectedId)**만 유지

```javascript
const [items, setItems] = useState(initialItems);
const [selectedId, setSelectedId] = useState(0);
```

- selectedId를 통해 선택된 항목을 고유하게 식별 가능
- 렌더링 시, items.find를 사용하여 선택된 항목을 계산하면, 항목이 수정되더라도 항상 최신 데이터를 사용가능

#### 이전 코드: 중복된 상태 관리

```javascript
const [items, setItems] = useState(initialItems);
const [selectedItem, setSelectedItem] = useState(items[0]);

<button onClick={() => setSelectedItem(item)}>Choose</button>;
```

items와 selectedItem이 중복되어, 데이터 동기화가 어렵습니다.
항목을 수정하면, selectedItem을 따로 업데이트해야 하는 문제가 있습니다.

#### 개선된 코드: selectedId로 식별

```javascript
const [items, setItems] = useState(initialItems);
const [selectedId, setSelectedId] = useState(0);

const selectedItem = items.find((item) => item.id === selectedId);

<button onClick={() => setSelectedId(item.id)}>Choose</button>;
```

- selectedItem은 렌더링 시 항상 최신 데이터로 계산됩니다.
- selectedId는 간단히 선택된 항목을 추적하며, 상태 관리가 간소화됩니다.

### (5) 깊게 중첩된 state 피하기

- 중첩된 객체는 업데이트가 어렵고 관리하기 어려움.
- state를 "평탄하게" 만들어 필요할 때 계산하도록 구성.
  State를 업데이트하기 쉽게 만들고 중첩된 객체의 다른 부분에 중복이 없도록 도와줍니다.

---

- 최종 목표: 간결하고 유지보수 가능한 구조를 통해 오류 가능성을 줄이고, 로직이 간단해지도록 설계.

---

## + Props를 state에 "미러링"하지 말아야 하는 이유

"미러링"이란 무엇인가?

"미러링"은 props를 state로 초기화하는 것

```javascript
function Message({ messageColor }) {
  const [color, setColor] = useState(messageColor);
}
```

- `color` state는` messageColor` prop으로 초기화됩니다.
- 하지만 prop 값이 변경되어도 state는 자동으로 업데이트되지 않음.
  - 예: 부모 컴포넌트에서 `messageColor`가 'blue'에서 'red'로 바뀌어도 `color`는 여전히 'blue' 상태로 유지.

### 왜 문제인가?

- 부모 컴포넌트의 messageColor 값과 자식 컴포넌트의 color state가 불일치하면 혼란을 초래.
  동기화 문제:
- color state는 첫 렌더링 이후에 prop의 변경 사항을 반영하지 않음.
- 두 소스에서 데이터를 관리하게 되어 버그 발생 가능성이 높아짐.

### 올바른 접근법: Props를 직접 사용

Props는 읽기 전용 데이터로 간주하며, 직접 사용하는 것이 가장 간단하고 안정적

```javascript
function Message({ messageColor }) {
  const color = messageColor; // props 직접 사용
}
```

- 부모 컴포넌트에서` messageColor`가 변경되면, 자식 컴포넌트는 자동으로 최신 값을 반영.
- 동기화 문제를 완전히 방지.

### 특정 상황에서 state로 "미러링"이 필요한 경우

props의 값이 변경되더라도 초기 값만 반영하고 싶을 때는 예외적으로 "미러링"을 사용가능

```javascript
function Message({ initialColor }) {
  const [color, setColor] = useState(initialColor); // 첫 번째 렌더링 시 초기화
}
```
