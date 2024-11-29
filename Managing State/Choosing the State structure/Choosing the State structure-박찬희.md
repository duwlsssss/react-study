# state 구조 선택하기

## state 구조화 원칙
state를 최적화 하지 않더라도 정상적인 프로그램을 작성 할 수 있지만, 오류가 발생할 확률을 줄이기 위해 state를 구조화하는 것이 좋고 다음과 같은 원칙을 가집니다.

## 1. 연관된 state 그룹화
```jsx
const [x, setX] = useState(0);
const [y, setY] = useState(0);
// 두 변수가 항상 같이 변경된다면, 단일 state로 통합!
const [position, setPosition] = useState({ x: 0, y: 0 });
```
위 예제는 마우스 커서의 움직임에 따라 x,y 좌표를 가져왔던 state입니다. 마우스 커서는 항상 x,y좌표를 가지며 두 값은 동시에 업데이트 됩니다. 따라서 `position`이라는 state로 통합할 수 있습니다.

필요한 state의 수를 모를 때, 데이터를 객체나 배열로 그룹화 하는 것도 유용합니다.
```jsx
const [position, setPosition] = useState({
   x: 0,
   y: 0,
   z: 0, //3차원 공간으로 바뀐다는 가정에 추가할 수 있음!
  });
```
이전 예제를 예시로 z축을 추가했지만, 예를 들어 주소를 예로 들면 어느 나라부터 시작할 것인지 도, 시, 읍, 면, 동이라는 기준을 객체에 추가할 수 있습니다.


## 2.state의 모순 피하기

```jsx
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
  ```
이번 예제는 text를 전송하는 버튼에 두가지 `isSending`과 `isSent`가 변경되는 상황입니다.
`handleSubmit`이벤트가 실행될 때, `isSending`과 `isSent`는 동시에 `true`가 되어서는 안되기 때문에 `typing`(초기값), `sending`,`sent` 세 가지 유효한 상태 중 하나를 가질 수 있는 `status` state변수로 대체하는 것이 좋습니다!

```jsx
  const [text, setText] = useState('');
  const [status, setStatus] = useState('typing');

  async function handleSubmit(e) {
    e.preventDefault();
    setStatus('sending');
    await sendMessage(text);
    setStatus('sent');
  }
```
## 3. 불필요한 state 피하기

