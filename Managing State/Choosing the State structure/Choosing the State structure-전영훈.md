# State 구조 선택하기

## State 구조화 원칙

### 연관된 state 그룹화하기

=>두 개 이상의 state 변수를 항상 동시에 업데이트한다면, 단일 state 변수로 병합하는 것을 고려
const [x, setX] = useState(0);
const [y, setY] = useState(0);
이런걸
const [position, setPosition] = useState({ x: 0, y: 0 }); 이렇게!
(State 변수가 객체인 경우에는 다른 필드를 명시적으로 복사하지 않고 하나의 필드만 업데이트할 수 없음)

### State의 모순 피하기

=>여러 state 조각이 서로 모순되고 “불일치”할 수 있는 방식으로 state를 구성하는 것은 실수가 발생할 여지를 만듭니다. 이를 피하세요.

setIsSending과 setIsSent라는 함수로 무언가를 할 때, 코드가 복잡해지면 sending과 동시에 sent를 하는것을 놓칠 가능성이 있다. 따라서 이러한 상태를 나타내는 코드는 const[status, setStatus]= useState('idle') //sent, sending로 표현해주는 것이 좋다.

### 불필요한 state 피하기

=>렌더링 중에 컴포넌트의 props나 기존 state 변수에서 일부 정보를 계산할 수 있다면, 컴포넌트의 state에 해당 정보를 넣지 않아야 합니다.

가령 firstName과 lastName의 state가 있는데 굳이 fullName이 필요할까?
이런건 그냥 firstName+lastName 의 값이 담긴 하나의 변수로 처리가 가능하다. 가능하면 state를 줄이자.

### State의 중복 피하기

=>가능하다면 중복을 줄이세요.

```jsx
const [items, setItems] = useState(initialItems);
const [selectedItem, setSelectedItem] = useState(items[0]);
```

위의 코드와 같이 여러 상태 변수 또는 중첩된 객체에서 동일한 데이터가 존재할 경우 동기화가 어려움. 가령 items의 내용 구성이 바뀌거나,

### 깊게 중첩된 state 피하기

=>여러개 중첩된 state는 업데이트시 변경된 부분부터 모든 객체의 복사본을 만들어야하기에 이 자체도 매우 어려우면서, 매우 어지러워짐 => 웬만하면 평탄화하는게 좋음
