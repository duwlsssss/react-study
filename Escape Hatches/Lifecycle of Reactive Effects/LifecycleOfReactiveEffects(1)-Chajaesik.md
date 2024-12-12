effects는 컴포넌트와 다른 생명 주기를 가진다.

컴포넌트는 마운트, 업데이트 또는 마운트 해제할 수 있다.

하지만 effect는 동기화를 시작하고 나중에 동기화를 중지하는 두 가지 작업만 할 수 있다.

 

### 학습내용
- effect의 생명주기가 컴포넌트의 생명주기와 다른 점
- 각 effect를 개별적으로 생각하는 방법
- effect를 다시 동기화해야 하는 시기와 그 이유
- effect의 의존성이 결정되는 방법
- 값이 유동적이라는 것의 의미
- 빈 의존성 배열이 의미하는 것
- React가 린터로 의존성이 올바른지 확인하는 방법
- 린터에 동의하지 않을 때 해야하는 일
 

### effect의 생명주기
모든 React 컴포넌트는 동일한 생명주기를 거친다.

```
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); //동기화 시작 
    connection.connect(); // 동기화 시작
    return () => {
      connection.disconnect(); //클린 업 함수
    };
  }, [roomId]);
  // ...
}
 ```

'React는 컴포넌트가 마운트될 때 동기화를 시작하고 컴포넌트가 마운트 해제될 때 동기화르 중지할 것'이라 생각할 수 있다.

하지만 때로는 컴포넌트가 마운트된 상태에서 동기화를 여러번 시작하고 중지해야할 수도 있다.

 

### 동기화가 두 번 이상 수행되어야 하는 이유
ChatRoom 컴포넌트가 사용자가 드롭다운에서 선택한 roomId prop으로 받는다고 가정해보자.
```
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId /* "general" */ }) { // 처음에 general 선택
  // ...
  return <h1>Welcome to the {roomId} room!</h1>;
}
 ```

UI가 표시되면 React가 effect를 실행하여 동기화를 시작한다.
```
    const connection = createConnection(serverUrl, roomId); // "general" 방에 연결
    connection.connect();
```
나중에 사용자가 드롭다운에서 다른 방(예: "trvel")을 선택한다면?

먼저 React가  UI를 업데이트한다.
```
function ChatRoom({ roomId /* "travel" */ }) {
  // ...
  return <h1>Welcome to the {roomId} room!</h1>;
}
 ```

그러면 어떻게 표시될까?

사용자는 UI에서 "travel"이 선택된 대화방임을 알 수 있겠다.

그러나 이전에 실행된 effect는 여전히 "general" 대화방에 연결되어있다.

roomId prop이 변경되었기 때문에 그때 effect가 수행한 작업("general" 방에 연결)이 더 이상 UI와 일치하지 않는다.

 

### 이 시점에서 React가 두 가지 작업을 수행하기를 기대한다.
1.  이전 roomId와의 동기화를 중지("general"방에서 연결 끊기)
2. 새 roomId와 동기화 시작("travel" 방에 연결)
우리가 배운 effect에서는 동기화를 시작하는 방법을 방금 알아보았고

이전에 배운 cleanup함수에는 동기화를 중지하는 방법을 알아봤었다.

이제 이것을 활용할 차례이다!

 

### React가 effect를 재 동기화하는 방법
동기화를 중지하기 위해 React는 "general"방에 연결한 후 effect가 반환한 cleanup함수를 호출한다.

현재 roomId가 general이었기 때문에 cleanup함수는 general 방에서 실행되어야한다.
```
function ChatRoom({ roomId /* "general" */ }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // "general" 방에 연결
    connection.connect();
    return () => {
      connection.disconnect(); // "general" 방에서 연결  클린업!
    };
 ```

이제 컴포넌트가 다른 roomId로 렌더링할 때마다 efffect가 다시 동기화된다.

예를들어 roomId가 travel에서 music으로 변경할 때, React는 다시 cleanup함수(travel 안에서)를 호출하여 effect 동기화를 중지시킨다.

마지막으로 사용자가 다른 화면으로 이동하면 ChatRoom이 마운트를 해제하여 effect 동기화를 중지한다.

### effect의 관점에서 생각하기
ChatRoom 컴포넌트의 관점에서 일어난 일을 요약해보면

1. roomId가 "general"으로 설정되어 마운트된 ChatRoom
2. roomId가 "travel"으로 설정되어 마운트된 ChatRoom
3. roomId가 "music"으로 설정되어 마운트된 ChatRoom
4. 마운트 해제된 ChatRoom

컴포넌트의 생명주기의 각 시점에서 effect는 아래의 작업을 수행했다,

1. effect가 "general" 대화방에 연결됨
2. "general"방에서 연결이 끊어지고 "treavel"방에 연결된 effect
3. "travel"방에서 연결이 끊어지고 "music" 방에 연결된 effect
4. "music" 방에서 연결이 끊어진 effect
### effect 자체의 관점에서 무슨 일이 발생했는가를 생각해보면
```
  useEffect(() => {
    // roomId로 지정된 방에 연결된 effect...
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      // ...연결이 끊어질 때까지
      connection.disconnect();
    };
  }, [roomId]);
  ```
항상 반 번에 하나의 시작/중지 사이클에만 집중해야 한다(효율적)

컴포넌트를 마운트, 업데이트 또는 마운트 해제하는 것은 중요하지 않다.

동기화를 시작하는 방법과 중지하는 방법만 작성하면 된다.

그렇다면 필요한 횟수만 effect를 시작하고 중지할 수 있는 탄력성을 확보할 수 있다!

 

#### React는 개발 단계에서 즉시 강제로 동기화를 수행하여 effect가 다시 동기화할 수 있는지 확인한다.

 

### React가 effect를 다시 동기화해야 한다는 것을 인식하는 방법
```
function ChatRoom({ roomId }) { // roomId prop은 시간이 지남에 따라 변경될 수 있습니다.
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // 이 effect는 roomId를 읽습니다.
    connection.connect();
    return () => {
      connection.disconnect();
    };
    
  }, [roomId]); // 따라서 React에 이 effect가 roomId에 "의존"한다고 알려줍니다.
  // ...
 ```

이전에 배웠듯, 의존성 배열에 roomId를 추가함으로써 상태 값이 변경되면 React가 다시 렌더링하도록 하였기 때문이다.

 

### 작동방식은 다음과 같다.

 

1. roomId가 prop이므로 시간이 지남에 따라 변경될 수 있다는 것을 알고 있다.
2. effect가 roomId를 읽는다는 것을 알고 있다.(따라서 로직이 나중에 변경될 수 있는 값에 따라 달라짐)
3. 그렇기 때문에 roomId를 effect의 종속성으로 지정한 것이다(roomId가 변경되면 다시 동기화 되도록)
컴포넌트가 다시 렌더링 될 때마다 React는 전달한 의존성 배열을 살펴본다.

배열의 값 중 하나라도 이전 렌더링 중에 전달한 동일한 지점의 값과 다르면 React는 effect를 다시 동기화한다.


### 각 effect는 별도의 동기화 프로세스를 나타낸다
단일 책임원칙에 따라
```
function ChatRoom({ roomId }) {
  useEffect(() => {
    logVisit(roomId);
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId]);
  // ...
} 
가 아닌 

function ChatRoom({ roomId }) {
  useEffect(() => {
    logVisit(roomId);
  }, [roomId]);

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    // ...
  }, [roomId]);
  // ...
}
 ```

코드의 각 effect는 별도의 독립적인 동기화 프로세스를 나타내야 한다.

 

따라서 코드를 살펴볼 때 코드가 더 깔끔해보이는지가 아니라 프로세스가 동일하거나 분리되어 있는지를 고려해야한다.

 

### 반응형 값에 "반응"하는 effect
effect에서 두 개의 변수(serverUrl, roomId)를 읽지만 종속성으로 roomId만 지정한 이유는

serverUrl의 값이 절대로 변경되지 않기 때문이다.

따라서 종속성으로 지정하는 것은 의미가 없다.

 

반면에 roomId는 다시 렌더링할 때 달라질 수 있따.

컴포넌트 내부에서 선언된 Props, state 및 기타값은 렌더링 중에 계산되고 React 데이터 흐름에 참여햐기 때문에 반응형이다.

 

 

만약 serverUrl을 반응형으로 만들고 싶다면 state로 선언한다.
```
function ChatRoom({ roomId }) { // Props는 시간이 지남에 따라 변화합니다.
  const [serverUrl, setServerUrl] = useState('https://localhost:1234'); // State는 시간이 지남에 따라 변화합니다.

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // effect는 Props와 state를 읽습니다.
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId, serverUrl]); // 따라서 이 effect는 props와 state에 "의존"한다고 React에 알려줍니다.
  // ...
}
```

serverUrl을 종속성으로  포함하면 effect가 변경된 후 다시 동기화되도록 할 수 있다