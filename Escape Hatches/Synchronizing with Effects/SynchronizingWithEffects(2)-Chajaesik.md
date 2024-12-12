### 3단계: 필요하다면 클린업을 추가하라
사용자가 표시될 때 채팅 서버에 연결해야 하는 ChatRoom  컴포넌트를 작성중이다.

createConnection() API가 주어지며, 이 API는 connect() 및 disconnect() 메서드를 가진 객체를 반환한다.

 

사용자에게 표시되는 동안 컴포넌트가 채팅 서버와의 연결을 유지하려면 어떻게 해야할까?

 

먼저 Effect를 작성하는데, 이때 매번 재렌더링 후에 채팅 서버에 연결하는 것은 느리므로 의존성  배열을 추가한다/

```
useEffect (() => {
	const connection = createConnection();
    connection.connect();
}, []);
 ```

Effect 내부의 코드는 어떠한 props나 상태를 사용하지 않으므로, 의존성 배열은 [](빈 배열)이다.

이는 React에게 이 코드를 컴포넌트가 "마운트"될 때만 실행하도록 알려준다.

즉 화면에 처음으로 나타날 때만 실행된다.

 

아래의 코드를 실행해보면 
```
import { useEffect } from 'react';
import { createConnection } from './chat.js';

export default function ChatRoom() {
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
  }, []);
  return <h1>채팅에 오신걸 환영합니다!</h1>;
}
```
[결과화면]

✅ 연결 중...

✅ 연결 중...

의존성 배열을 [] 빈 배열로 추가했으므로, 마운트 될 때만 실행될 것이고, 이는 ✅ 연결 중... 가 한번 출력될 것으로 에상했는데  두번 출력되었다.

## 왜일까?
ChatRoom 컴포넌트가 여러 화면으로 구성된 큰 앱의 일부라고 가정해보자

컴포넌트가 마운트되고 connection.connect()를 호출한다.

그런 다음 사용자가 예를 들어 설정 페이지 같은 다른 페이지로 이동한다면 ChatRoom 컴포넌트가 마운트 해제된다.

마지막으로 사용자가 뒤로 가기 버튼을 클릭하고 ChatRoom이 다시 마운트된다.

이렇게 되면 두 번째 연결이 설정되지만 첫번째 연결은 종료되지 않은 것을 기억하자

이러한 경험으로 사용자가 앱을 탐색하는 동안 연결은 종료되지 않고 계속 쌓일 것이다.

 

이와같은 버그는 앱의 이곳저곳을 수동으로 테스트해보지 않으면 놓치기 쉽다.

이러한 문제를 빠르게 파악할 수 있도록 React는 개발 모드에서 초기 마운트 후 모든 컴포넌트를 한번 다시 마운트한다.

 

컴포넌트가 마운트 해제될 때 연결을 닫지 않는 문제를 해결하려면 Effect에서 "클린업 함수"를 반환하면 된다.
```
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    return () => {
      connection.disconnect(); // 클린 업! 함수
    };
  }, []);
  ```
이제 개발 모드에서 세 개의 콘솔을 확인할 수 있다.


마운트가 해제되도 다시보자!
 

이것이 개발 모드에서 올바른 동작이다.

 

### 개발 중에 Effect가 두 번 실행되는 경우를 다루는 방법
React는 방금 전과 같은 버그를 찾기 위해 개발 중에 컴포넌트를 명시적으로 다시 마운트 한다고 하였다

 

"Effect를 한 번 실행하는 방법이 아닌 어떻게 Effect가 다시 마운트된 후에도 작동하도록 고칠 것인가"가 올바른 질문이다.

 

일반적으로 정답은 클린업 함수를 구현하는 것이다.

클린업 함수는 Effect가 수행하던 작업을 중단하거나 되돌리는 역할을 수행한다.

기본 원칙은 사용자가 Effect가 한 번실행되는 것 (배포 환경과 같이) 설정-> 클린업 -> 설정 순서간에 차이를 느끼지 못해야 한다.

 

### !주의! Effect가 두 번 실행되는 것을 막기위해 ref를 사용하지 마세요
아래와 같이 useRef를 사용하여 "수정"하려고 할 수 있다.
```
  const connectionRef = useRef(null);
  useEffect(() => {
    // 🚩 버그를 수정하지 않습니다!!!
    if (!connectionRef.current) {
      connectionRef.current = createConnection();
      connectionRef.current.connect();
    }
  }, []);
  ```
이렇게 하면 개발 모드에서 "✅ 연결 중..."이 한 번만 출력되지만 버그가 수정된 것은 아니다.

사용자가 다른 곳에 가더라도 연결이 끊어지지 않고 사용자가 돌아왔을 때 새로운 연결이 생성되므로 계속 쌓이게 된다.

Effect는 위에 있는 예시가 연결을 클린업 한것 처럼 다시 마운트된 이후에도 제대로 동작해야 한다.

즉 Effect가 다시 마운트로 인해 중단된 경우 클린업 함수를 구현해야 한다.

 

따라서 작성할 대부분의 Effect는 아래의 일반적인 패턴 중 하나에 해당할 것이다.
 

### 1. React로 작성되지 않은 위젯 제어하기
React로 작성하지 않은 UI 위젯을 추가해야할 때, 예를 들어 페이지에 지도 컴포넌트를 추가한다고 가정해보자.

이 지도 컴포넌트에는 setZoomLevel() 메서드가 있으며, zommLevel state변수와 동기화하려고 할 것이다.
```
useEffect(() => {
  const map = mapRef.current;
  map.setZoomLevel(zoomLevel);
}, [zoomLevel]);
 ```

이 경우에는 클린업이 필요하지 않다

왜? --> 개발 모드에서 React는 Effect를 두 번 호출하지만, 동일한 값을 가지고 setZoomLevel을 두 번 호출하는 것은 아무런 문제가 되지 않는다.

 

하지만 일부 API는 연속해서 두 번 호출하는 것을 허용하지 않을 수 있다.

예를 들어 내장된 `<dialog>`요소의 showModal 메서드는 두번 호출하면 예외를 던지는데,

클린업 함수로 이 함수에서 대화 상자를 닫히도록 할 수 있다.
```
useEffect(() => {
  const dialog = dialogRef.current;
  dialog.showModal();
  return () => dialog.close(); // 클린업 함수
}, []);
Effect가 showModal()을 호출한 다음, 즉시 close()를 호출하고 다시 showModal()을 호출한다.
```
 

아래의 예시들은 이와 같은 내용이다.

### 이벤트 구독하기

```
useEffect(() => {
  function handleScroll(e) {
    console.log(window.scrollX, window.scrollY);
  }
  window.addEventListener('scroll', handleScroll);
  return () => window.removeEventListener('scroll', handleScroll);
}, []);
```
이것은 제품 환경에서 한번 addEventListener()를 호출하는 것과 동일한 로직이다.

 

## 애니메이션 트리거
```
useEffect(() => {
  const node = ref.current;
  node.style.opacity = 1; // Trigger the animation
  return () => {
    node.style.opacity = 0; // Reset to the initial value
  };
}, []);
 ```

### 데이터 패칭
```
useEffect(() => {
  let ignore = false;

  async function startFetching() {
    const json = await fetchTodos(userId);
    if (!ignore) {
      setTodos(json);
    }
  }

  startFetching();

  return () => {
    ignore = true;
  };
}, [userId]);
```
이미 발생한 네트워크 요청을 "실행 취소"할수는 없지만, 클린업 함수는 더이상 관련이 없는 패치가 애플리케이션에 계속 영향을 미치지 않도록 보장해야 한다.

userId가 'Alice"에서 "Bob"으로 변경되면 클린업은 "Bob" 이후에 도착하더라도 "Alice" 응답을 무시하도록 보한다.

 

### 좋은 코드 예시..
```
function TodoList() {
  const todos = useSomeDataLibrary(`/api/user/${userId}/todos`);
 ```

 

React는 항상 이전 렌더의 Effect를 제거하고 다음 렌더의 Effect를 실행한다.