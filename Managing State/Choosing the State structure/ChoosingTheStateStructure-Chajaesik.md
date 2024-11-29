# State를 구조화하면 수정과 디버깅이 즐거워집니다!
 

### 학습 내용
- 단일 vs 다중 state 변수를 사용하는 경우
- State를 구성할 때 피해야할 사항
- 상태 구조의 일반적인 문제를 해결하는 방법

### State 구조화 원칙
#### 상태를 갖는 구성요소를 작성할 때, 사용할 state 변수의 수와 데이터 형태를 정해야 한다.

### 연관된 state 그룹화하기
- 두 개 이상의 state 변수를 항상 동시에 업데이트 한다면, 단일 state 변수로 병합
### State의 모순 피하기 
-  여러 state 조각이 서로 모순되고 "불일치"할 경우를 피하라
### 불필요한 state 피하기
- 렌더링 중에 컴포넌트의 props나 기존 state 변수에서 일부 정보를 계산할 수 있다면, 컴포넌트의 state에 해당 정보를 넣지 않아야 한다.

### State의 중복 피하기
- 여러 상태 변수 또는 중첩된 객체에서 동일한 데이터가 존재할 경우 동기화 유지가 어렵다
- 깊게 중첩된 state피하기
- 가능하면 state를 평탄화 방식으로 구성

#### 위의 원칙들을 통해 "오류 없이 상태를 쉽게 업데이트"할 수 있다.

 

### 연관된 state 그룹화하기
 

#### 아래의 코드 예시를 보자

 
```javascript
const [x, setX] = useState(0);
const [y, setY] = useState(0);

// 아니면 이렇게?

const [position, setPosition] = useState({ x:0, y:0 })
 ```

#### 기술적으로 두가지 방법 모두 사용할 수 있지만, 두 개의 state 변수가 항상 함께 변경된다면 단일 state 변수로 통합하는 것이 좋다.

 
```javascript
  const [position, setPosition] = useState({ // 객체를 활용
    x: 0,
    y: 0
  });
  
  // 생략...
  
   setPosition({
      x: e.clientX,
      y: e.clientY
   });
 ```

### !주의 !

#### State 변수가 객체인 경우에는 다른 필드를 명시적으로 복사하지 않고, 하나의 필드만 업데이트 할 수 없다.

 
```javascript
// 예를 들어..

setPosition({ x: 100 })은 y 속성이 존재하지 않기 때문에 사용할 수 없다

// x만 설정하려면...

setPosition({ ...position, x: 100})

또는 

useState 두개를 써서 setX(100)을 해야한다
 ```

 

### State의 모순 피하기
#### 아래의 코드를 보자..

 
```javascript
export default function FeedbackForm() {
  const [text, setText] = useState('');
  const [isSending, setIsSending] = useState(false);
  const [isSent, setIsSent] = useState(false);

  async function handleSubmit(e) {
    e.preventDefault();
    setIsSending(true);
    await sendMessage(text);
    setIsSending(false);
    setIsSent(true);
  }
}
 ```

#### `handleSubmit`부분에서, `setIsSent`와 `setIsSending`을 함께 호출하지 않았을 때에도 값이 둘 다 true로 변한다.

#### 컴포넌트가 복잡할 수록 위와같이 의도하지 않은 일이 발생하기 쉽고, 이해하기 어렵다

 
#### isSending과 insSent는 동시에 true가 되어서는 안되기 때문에, 

#### 이 두 변수를 'typing'(초기값), 'sending', 'sent' 세 가지 유효한 상태중 하나를 가질 수 있는 status state 변수로 대체하는 것이 좋다.

 
```javascript
export default function FeedbackForm() {
  const [text, setText] = useState('');
  const [status, setStatus] = useState('typing');

  async function handleSubmit(e) {
    e.preventDefault();
    setStatus('sending');
    await sendMessage(text);
    setStatus('sent');
  }

  const isSending = status === 'sending';
  const isSent = status === 'sent';
 ```
#### 불필요한 state 피하기 않아야 한다.

### State의 중복 피하기

- 여러 상태 변수 또는 중첩된 객체에서 동일한 데이터가 존재할 경우 동기화 유지가 어렵다

### 깊게 중첩된 state피하기

- 가능하면 state를 평탄화 방식으로 구성

#### 위의 원칙들을 통해 "오류 없이 상태를 쉽게 업데이트"할 수 있다.

 
### 불필요한 state 피하기
#### 성과 이름을 합치는 아래의 코드를 살펴보자 

```javascript
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');
  const [fullName, setFullName] = useState('');

  function handleFirstNameChange(e) {
    setFirstName(e.target.value);
    setFullName(e.target.value + ' ' + lastName);
  }

  function handleLastNameChange(e) {
    setLastName(e.target.value);
    setFullName(firstName + ' ' + e.target.value);
  }
 ```

#### 각 function 안에서 setFullName을 호출하여, fullName의 값을 변경하고 있다.

 

### 하지만 꼭 function안에서 갖고올 필요는 없다.

 
```javascript
export default function Form() { // 렌더링 될 때마다 호출
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');

  const fullName = firstName + ' ' + lastName
  }
 ```

#### setFirstName 또는 setLastName을 호출하면 렌더링이 진행되는데,
####  이때 Form()은 다시 위에서부터 아래로 코드를 읽어 실행하기 때문에 fullName의 값을 구할 수 있다.

 

### Props를 state에 미러링하지 마세요!
 

#### 다음은 불필요한 state의 일반적인 에이다.


```javascript
function Message({ messageColor }) { // 그 이후 초기화
  const [color, setColor] = useState(messageColor); // 첫번째 렌더링에만 초기화
 ```

#### 여기서 color state변수는 messageColor prop으로 초기화 된다.

#### 나중에 부모 컴포넌트가 다른 값의 messageColor를 전달한다면 color state 변수는 업데이트되지 않는다.

 

### 왜냐하면 ~

 

#### State는 첫번쨰 렌더링 중에만 초기화되기 때문이다.

 

#### 따라서, 아래와 같이 작성하여 부모 컴포넌트에서 전달된 prop와 동기화를 잃지 않을 수 있다.

```javascript
function Message({ messageColor }) {
  const color = messageColor;
 ```

### 그렇다면 미러링은 언제 사용할까 ?

 

#### Props를 상태로 "미러링"하는 것은 특정 prop에 대한 모든 업데이트를 무시하기를 원할 때만 사용

#### 관례에 따라 prop의 이름을 initial 또는 default로 시작하여 새로운 값이 무시됨을 명확히 하자

 

 

### State의 중복 피하기
#### state는 다음과 같이 중복되어 있다.

```javascript
items = [{ id: 0, title: 'pretzels'}, ...
selecteditem = {id: 0, title: 'pretzels'}
```
#### 하지만 변경 된 후는 다음과 같다.

 
```javascript
items = [{ id: 0, title: 'pretzels'}, ...}
selectedId = 0
```
#### 중복은 사라지고 필수적인 state만 유지된다!

### 깊게 중첩된 state 피하기
 

#### 다음과 같은 초기여행계획이 있다. 

#### 우선 4개의 지역을 방문하기로 했었는데... 

```javascript
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
 ```

#### 갑자기 마음이 바뀌어 Botswana를 안 가려고 하여 장소를 삭제하려 한다.

 

#### 우선 우리가 배운 filter를 이용하여 삭제하려 할 수 있다.

 

#### 이때 주의할 점은 원본 객체의 값은 불변성을 가진다에 따라 filter는 새로운 배열을 복사할텐데 하나의 장소를 

#### 삭제하기 위해서 모든 배열을 복사한다는 것은 매우 비효율 적이다.( 만약 1000개라면..?)

 

#### 따라서 state가 쉽게 업데이트하기에 너무 "중첩"되어 있다면 "평탄"하게 만들어 보자!

 

#### 이때 주의해야할 점은 각 place가 자식 장소의 배열을 가지는 트리구조(기존)대신, 
#### 각 장소가 자식 장소ID의 배열을 가질 수 있도록 하는 것이다.
#### 그 다음 각 장소 ID와 해당 장소에 대한 매핑을 저장한다.

 
```javascript
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
 ```

#### 이제 state가 "평탄(정규화)"하므로 중첩된 업데이트가 더 쉬워졌다!

 

#### 이제 장소를 제거하기 위해, 다음의 두 단계를 업데이트 하면된다.


- 업데이트된 버전의 부모장소는 childIds 배열에서 제거된 ID를 제외해야 한다.
- 업데이트된 버전의 루트 "테이블"객체는 부모 장소의 업데이트된 버전을 포함해야 한다.
```javascript
function handleComplete(parentId, childId) {
    const parent = plan[parentId];
    // Create a new version of the parent place
    // that doesn't include this child ID.
    const nextParent = {
      ...parent,
      childIds: parent.childIds
        .filter(id => id !== childId)
    };
    // Update the root state object...
    setPlan({
      ...plan,
      // ...so that it has the updated parent.
      [parentId]: nextParent
    });
  }
```