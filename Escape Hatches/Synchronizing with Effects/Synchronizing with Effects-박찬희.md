# Effect로 동기화하기
일부 컴포넌트에서 외부 시스템과 동기화해야 할 수 있습니다. Effect를 사용하면 랜더링 후 특정 코드를 실행하여 React 외부의 시스템과 컴포넌트를 동기화 할 수 있습니다.

## Effect란 무엇이고 이벤트와는 어떻게 다른가요?

컴포넌트 내부에는 2가지 로직 유형이 있습니다.
- **랜더링 코드**를 주관하는 로직은 컴포넌트의 최상단에 위치하며, props와 state를 적절히 변형해 결과적으로 JSX를 반환합니다. 랜더링 코드 로직은 순수해야합니다.

- **이벤트 핸들러**는 단순한 계산 용도가 아닌 무언가를 하는 컴포넌트 내부의 중첩 함수입니다. 이벤트 핸들러는 입력 필드를 업데이트하거나, 제품을 구입하기 위해 HTTP POST 요청을 보내거나, 사용자를 다른 화면으로 이동시킬 수 있습니다. 

가끔은 이것으로 충분하지 않습니다. 화면에 보일 때마다 채팅 서버에 접속해야 하는 `ChatRoom`컴포넌트를 생각해봅시다. 서버에 접속하는 것은 순수한 계산이 아니고 부수 효과(Side Effect)를 발생시키기 때문에 랜더링 중에는 할 수 없습니다. 하지만 클릭 한 번으로 `ChatRoom`이 표시되는 특정 이벤트는 하나도 없습니다.

Effect는 랜더링 자체에 의해 발생하는 부수 효과를 특정하는 것으로 특정 이벤트가 아닌 랜더링에 의해 직접 발생합니다. 채팅에서 메시지를 보내는 것은 이벤트입니다. 왜냐하면 이것은 사용자가 특정 버튼을 클릭함에 따라 직접적으로 발생합니다. 그러나 서버 연결 설정은 Effect입니다. 이것은 컴포넌트의 표시를 주관하는 어떤 상호 작용과도 상관없이 발생해야 합니다. Effect는 커밋이 끝난 후에 화면 업데이트가 이루어지고 나서 실행됩니다. 이 시점에 React 컴포넌트를 외부 시스템과 동기화하기 좋은 타이밍입니다.

## Effect가 필요 없으지도 모릅니다

컴포넌트에 Effect를 무작정 추가하지 마세요. Effect는 주로 React 코드를 벗어난 특정 외부 시스템과 동기화 하기 위해 사용됩니다. 이는 브라우저 API, 서드 파티 위젯, 네트워크 등을 포험합니다. 만약 당신의 Effect가 단순히 다른 상태에 기반하여 일부 상태를 조정하는 경우 Effect가 필요하지 않을 수 있습니다.

## Effect를 작성하는 법

Effect를 작성하기 위해서는 다음 세 단계를 따릅니다.

1. Effect 선언 - 기본적으로 Effect는 모든 커밋 이후에 실행됩니다.
2. Effect 의존성 지정 - 대부분의 Effect는 모든 랜더링 후가 아닌 필요할 때만 다시 실행되어야 합니다.
예를들어 페이드 인 애니메이션은 컴포넌트가 나타날 때에만 트리거되어야 합니다. 채팅방에 연결, 연결 해제하는 것은 컴포넌트가 나타나거나 사라질 때 또는 채팅 방이 변경될 때만 발생해야 합니다.
3. 필요한 경우 클린업 함수 추가. 일부 Effect는 수행 중이던 작업을 중지, 취소 또는 정리하는 방법을 지정해야 할 수 있습니다. 

### 1단계: Effect 선언하기

컴포넌트 내에서 Effect 선언하기 위해서 React에서 `useEffect` Hook을 import하고 컴포넌트 최상위 레벨에서 호출하고 Effect 내부에 코드를 넣어야합니다.
```jsx
import { useEffect } from 'react';

function MyComponent() {
  useEffect(() => {
    // 이곳의 코드는 *모든* 렌더링 후에 실행됩니다
  });
  return <div />;
}
```
컴포넌트가 랜더링 될 때마다 React는 화면을 업데이트한 다음 `useEffect`내부의 코드를 실행합니다.
즉 `useEffect`는 화면에 랜더링이 반영될 때까지 코드 실행을 지연시킵니다.

이제 외부 시스템과 동기화하기 위해 어떻게 Effect를 사용할 수 있는지 알아보겠습니다. `<VideoPlayer>`라는 React 컴포넌트를 살펴보겠습니다. 이 컴포넌트를 `isPlaying`이라는 props를 통해 재생 중인지 일시 정지 상태인지 제어하는 것이 좋아보입니다.
```jsx
<VideoPlayer isPlaying={isPlaying} />;
```

커스텀 `VideoPlayer` 컴포넌트는 내장 브라우저 `<video>`태그를 랜더링합니다.
```jsx
function VideoPlayer({ src, isPlaying }) {
  // TODO: isPlaying을 활용하여 무언가 수행하기
  return <video src={src} />;
}
```
`<video>` 태그에는 `isPlaying` prop이 없습니다. 이를 제어하는 유일한 방법은 DOM 요소에서 수동으로 `play()` 및 `pause()` 메소드를 호출하는 것입니다. `isPlaying` prop의 값을 `play()` 및 `pause()`와 같은 호출과 동기화해야합니다.

먼저 `<video>` DOM 노드의 ref를 가져와야 합니다.
```jsx
import { useState, useRef, useEffect } from 'react';

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  if (isPlaying) {
    ref.current.play();  // 렌더링 중에 이를 호출하는 것이 허용되지 않습니다.
  } else {
    ref.current.pause(); // 역시 이렇게 호출하면 바로 위의 호출과 충돌이 발생합니다.
  }

  return <video ref={ref} src={src} loop playsInline />;
}

export default function App() {
  const [isPlaying, setIsPlaying] = useState(false);
  return (
    <>
      <button onClick={() => setIsPlaying(!isPlaying)}>
        {isPlaying ? '일시정지' : '재생'}
      </button>
      <VideoPlayer
        isPlaying={isPlaying}
        src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
      />
    </>
  );
}
```
이 코드가 올바르지 않은 이유는 랜더링 중에 DOM노드를 조작하려고 시도하기 때문입니다. React에서 랜더링이 JSX의 순수한 계산이여야 하며, DOM 수정과 같은 부수 효과를 포함해서는 안됩니다.

게다가, 처음으로 `VideoPlayer`가 호출될 떄 해당 DOM이 아직 존재하지 않습니다. React는 컴포넌트가 JSX를 반환할 때까지 어떤 DOM을 생성할지 모르기 때문에 `play()` 또는 `pause()`를 호출할 DOM 노드가 아직 없습니다.

해결책은 부수 효과를 랜더링 연산에서 분리하기 위해 `useEffect`로 감싸는 것입니다.

```jsx
import { useEffect, useRef } from 'react';

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {
    if (isPlaying) {
      ref.current.play();
    } else {
      ref.current.pause();
    }
  });

  return <video ref={ref} src={src} loop playsInline />;
}
```

DOM 업데이트를 Effect로 감싸면 React가 화면을 업데이트한 다음에 Effect가 실행됩니다.
`VideoPlayer` 컴포넌트가 랜더링 될 때 몇가지 일이 발생합니다. 먼저 React는 화면을 업데이트하여 `<video>`태그가 올바른 속성과 함께 DOM에 있는지 확인합니다. 그런 다음 React는 Effect를 실행합니다. 마지막으로 Effect에서는 `isPlaying`값에 따라 `play()`또는 `pause()`를 호출합니다.

```jsx
import { useState, useRef, useEffect } from 'react';

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {
    if (isPlaying) {
      ref.current.play();
    } else {
      ref.current.pause();
    }
  });

  return <video ref={ref} src={src} loop playsInline />;
}

export default function App() {
  const [isPlaying, setIsPlaying] = useState(false);
  return (
    <>
      <button onClick={() => setIsPlaying(!isPlaying)}>
        {isPlaying ? '일시 정지' : '재생'}
      </button>
      <VideoPlayer
        isPlaying={isPlaying}
        src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
      />
    </>
  );
}
```

이 예시에서 React 상태와 동기화 된 외부 시스템은 브라우저 미디어 API였습니다. 이와 비슷한 접근 방식으로 React가 아닌 레거시 코드(ex:jQuery 플러그인)를 선언적인 React 컴포넌트로 감싸는 데에도 사용할 수 있습니다.

실제로 비디오 플레이어를 제어하는 것은 훨씬 복잡합니다. `play()`를 호출하는 것이 실패할 수 있으며, 사용자는 컴포넌트의 UI가 아닌 브라우저 내장 컨트롤을 사용하여 동영상을 재생 또는 일시 정지할 수 있습니다. 

> 주의!
기본적으로 Effect는 모든 랜더링 이후 실행됩니다. 이러한 이유로 아래와 같은 코드는 무한 루프를 만들어냅니다.
```jsx
const [count, setCount] = useState(0);
useEffect(() => {
  setCount(count + 1);
});
```
위 예제는 Effect로 state를 설정하면서 리랜더링 시키고 있습니다. 그럼 다시 Effect가 실행되고 그러면 또 다시 state가 설정되며 리랜더링 되어 무한 반복됩니다.

Effect는 일반적으로 컴포넌트 외부 시스템과 동기화하는 데 사용됩니다. 외부 시스템이 없고 다른 상태에 기반하여 상태를 조정하려는 경우네느 Effect가 필요하지 않을 수 있습니다.

### 2단계: Effect의 의존성 지정하기

기본적으로 Effect는 모든 랜더링 후 실행됩니다. 어떤 때에는 랜더링 후 실행되는 것을 원하지 않을 수도 있습니다.
- 외부 시스템과 동기화하는 것이 항상 즉시 이루어지지 않그 때문에 필요하지 않을 경우에는 실행을 건너 뛰고 싶을 수 있습니다.
- 모든 키 입력마다 컴포넌트 fade-in 애니매이션을 트리거하기를 원하지 않을 것입니다. 애니메이션은 컴포넌트가 처음 나타날 때에만 한 번 실행되어야 합니다.

다음 예제는 위 문제들을 설명하기 위한 예시로 이전 예시에 부모 컴포넌트의 상태를 업데이트하는 텍스트 입력을 추가했습니다.

```jsx
import { useState, useRef, useEffect } from 'react';

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {
    if (isPlaying) {
      console.log('video.play() 호출');
      ref.current.play();
    } else {
      console.log('video.pause() 호출');
      ref.current.pause();
    }
  }); // }, [isPlaying]); 이런 식으로 의존성 배열을 추가하게 되면 매번 호출되지 않을 것입니다.

  return <video ref={ref} src={src} loop playsInline />;
}

export default function App() {
  const [isPlaying, setIsPlaying] = useState(false);
  const [text, setText] = useState('');
  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <button onClick={() => setIsPlaying(!isPlaying)}>
        {isPlaying ? '일시 정지' : '재생'}
      </button>
      <VideoPlayer
        isPlaying={isPlaying}
        src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
      />
    </>
  );
}
```

해당 예시는 video가 재생되고 있다면 계속해서 `play`메소드를 호출하고 반대의 경우라면 `pause`메소드를 호출하고 있습니다.
React에게 Effect를 불필요하게 다시 실행하지 않도록 지시하려면 `useEffect`호출의 두 번째 인자로 의존성 배열을 지정해야합니다.
위 예시에서는 `[isPlaying]`배열을 의존성 배열로 지정하게 된다면 React에게 이전 랜더링 중에 `isPlaying`이 이전과 동일하다면 다시 실행하지 않도록 해야한다고 알려주고 계속 호출되지 않을것입니다.
이렇게되면 재생/일시 정지 버튼을 누를때만 Effect가 실행될 것입니다.

React가 지정한 모든 종속성이 이전 랜더링의 것과 정확히 동일한 값을 가진 경우에만 Effect는 다시 실행되지 않습니다.
의존성을 선택할 수 없다는 것에 유의해야합니다. 
의존성배열에 지정한 종속성이 Effect 내부의 코드를 기반으로 React가 기대하는 것과 일치하지 않으면 린트 에러가 발생합니다.
> 의존성 배열이 없는 경우와 빈 배열인 경우 동작이 다릅니다.
```jsx
useEffect(() => {
  // 모든 렌더링 후에 실행됩니다
});

useEffect(() => {
  // 마운트될 때만 실행됩니다 (컴포넌트가 나타날 때)
}, []);

useEffect(() => {
 // 마운트될 때 실행되며, *또한* 렌더링 이후에 a 또는 b 중 하나라도 변경된 경우에도 실행됩니다
}, [a, b]);
```

### 3단계: 필요하다면 클린업을 추가하세요

사용자에게 표시될 때 채팅 서버에 연결해야하는 `ChatRoom`컴포넌트를 작성중이라고 생각해봅시다.
`createConnection()` API가 주어지며, 이 API는 `connect()` 및 `disconnect()` 메소드를 가진 객체를 반환합니다.
사용자에게 표시 되는 동안 컴포넌트가 채팅 서버와 연결을 유지하려면 Effect를 사용해야합니다.

```jsx
// App.js
import { useEffect } from 'react';
import { createConnection } from './chat.js';

export default function ChatRoom() {
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
  }, []); // 매번 재랜더링 후 채팅 서버에 연결하는 것은 느리므로 의존성 배열을 추가(컴포넌트가 나타날 때만 서버와 연결)
  return <h1>채팅에 오신걸 환영합니다!</h1>;
}

// chat.js
export function createConnection() {
  // 실제 구현은 정말로 채팅 서버에 연결하는 것이 되어야 합니다.
  return {
    connect() {
      console.log('✅ 연결 중...');
    },
    disconnect() {
      console.log('❌ 연결이 끊겼습니다.');
    }
  };
}
```
이 Effect는 마운트 될때만 실행되므로 '✅ 연결 중...'이 콘솔에 한 번만 출력되는 것을 예상할 수 있지만 확인해보면 두 번 출력됩니다.

`ChatRoom`컴포넌트가 여러 화면으로 구성된 큰 앱의 일부라고 가정했을 때, 사용자가 ChatRoom 페이지에서 여정을 시작합니다.
컴포넌트가 마운트되고 `connection.connect()`를 합니다.
그런 다음 사용자가 다른 화면으로 이동할 때 ChatRoom 컴포넌트가 마운트 해제됩니다.
마지막으로 사용자가 뒤로 가기 버튼을 클릭하고 ChatRoom이 다시 마운트 됩니다.
이러면 두 번째 연결이 설정되지만 첫 연결은 아직 종료되지 않았습니다. 이는 사용자가 다른 화면으로 갔다 돌아올 수록 쌓일 것입니다.

이 문제는 Effect에서 마운트 해제될 때 연결을 닫지 않는 것에서 시작되었습니다.
해결을 위해서는 Effect에서 클린업 함수를 반환하면 됩니다.

React는 Effect가 다시 실행되지 전마다 클린업 함수를 호출하고, 컴포넌트가 마운트 해제될 때에도 마지막으로 호출합니다.
클린업 함수가 구현된 경우 어떤 일이 일어나는지 살펴보겠습니다.
```jsx
//App.js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

export default function ChatRoom() {
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    return () => connection.disconnect(); // 클린업 함수 추가
  }, []);
  return <h1>채팅에 오신걸 환영합니다!</h1>;
}
```
이번 예제에서는 콘솔로그에서 연결 이후 
✅ 연결 중...
❌ 연결 해제됨
✅ 연결 중...
이와 같은 콘솔 창을 확인 할 수 있습니다.
연결을 해제하고 다시 연결함으로써 사용자가 다른 부분을 탐색하고 다시 돌아와도 코드가 깨지지 않을 것입니다.
클린업을 잘 구현하기 위해서는 Effect를 한 번 실행하는 것과 실행, 클린업, 이후 다시 실행하는 것 사이에 사용자에게 보이는 차이가 없어야합니다.

## 개발 중에 Effect가 두 번 실행되는 경우를 다루는 방법

React는 위에서 다뤘던 예시와 같은 버그를 찾기 위해 개발 중에 컴포넌트를 명시적으로 다시 마운트합니다. 
Effect를 한 번 실행하는 방법이 아니라 어떻게 Effect가 다시 마운트 된 후에도 작동하도록 고칠 것인가를 찾기 위해서 입니다.

그리고 이 해답은 일반적으로 클린업 함수를 구현하는 것입니다. 클린업 함수는 Effect가 수행하던 작업을 중단하거나 되돌리는 역할을 합니다.
기본 원칙은 Effect가 한 번 실행되는 것과 설정 -> 클린업 -> 설정 순서간에 차이를 느끼지 못해야 합니다.

### React로 작성되지 않은 위젯 제어하기

다음 예시는 페이지에 지도 컴포넌트를 추가하는 상황입니다. 
지도 컴포넌트에는 `setZoomLevel()`메소드가 있으며, `zoomLevel` state 변수와 동기화하려고 할 것입니다.
```jsx
useEffect(() => {
  const map = mapRef.current;
  map.setZoomLevel(zoomLevel);
}, [zoomLevel]);
```
이 경우에는 클린업이 필요하지 않습니다. 
React는 개발모드에서 Effect를 두번 호출하지만 동일 한 값을 가지고 `setZoomLevel`을 두번 호출하는 것은 문제가 되지 않기 때문입니다.
일부 API는 두 번 호출되는 것을 허용하지 않을 수 있습니다.
내장된 `<dialog>`요소의 `showModal` 메소드는 두 번 호출하면 예외를 던집니다.
모달이 두번 열리는 것은 정상적이지 않기 때문이죠
그렇기 때문에 클린업 함수를 구현하고 대화상자를 닫을 수 있도록 해야합니다.
```jsx
useEffect(() => {
  const dialog = dialogRef.current;
  dialog.showModal();
  return () => dialog.close();
}, []);
```

### 이벤트 구독하기

만약 Effect가 어떤 것을 구독한다면 클린업함수에서 구독을 해지해야합니다.
```jsx
useEffect(() => {
  function handleScroll(e) {
    console.log(window.scrollX, window.scrollY);
  }
  window.addEventListener('scroll', handleScroll);
  return () => window.removeEventListener('scroll', handleScroll);
}, []);
```
개발 중에는 Effect가 `addEventListener()`를 호출한 다음 즉시 `removeEventListener()`를 호출하고, 그 다음 동일한 핸들러로 `addEventListener()`를 호출합니다.
따라서 한 번에 하나의 활성 구독만 있게 됩니다. 이것은 제품 환경에서 한 번 `addEventListener()`를 호출하는 것과 동일한 동작을 가집니다.

### 애니메이션 트리거되어야
Effect가 어떤 요소를 애니메이션으로 표시하는 경우, 클린업 함수에서 애니메이션을 초기값으로 재설정 해야합니다.
```jsx
useEffect(() => {
  const node = ref.current;
  node.style.opacity = 1; // 애니메이션 트리거
  return () => {
    node.style.opacity = 0; // 초기값으로 재설정
  };
}, []);
```
개발 중에는 불투명도가 `1`로 설정되고, 그런 다음 `0`으로 설정 된 후, 다시 `1`로 설정 됩니다.
이것은 제품환경에서 `1`로 직접 설정하느 ㄴ것과 동일한 동작을 가집니다.

### 데이터 페칭

Effect가 어떤 데이터를 가져온다면, 클린업 함수에서는 fetch를 중단하거나 결과를 무시해야합니다.
```js
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
이미 발생한 네트워크 요청을 실행 취소할 수는 없지만, 클린업 함수에서 더 이상 관련이 없는 fetch가 애플리케이션에 계속 영향을 미치지 않도록 보장해야합니다.
`userId`가 `Alice`에서 `Bob`으로 변경되면 클린업은 `Bob`이후에 도착하더라도 `Alice`응답을 무시하도록 보장합니다.

### 분석 보내기

페이지 방문 시 분석 이벤트를 보내는 코드입니다.
```js
useEffect(() => {
  logVisit(url); // POST 요청을 보냄
}, [url]);
```
개발 환경에서는 `logVisit`가 각 URL에 대해 두 번 호출될 것입니다. 그래서 이를 수정하고자 할 수 있지만 이 코드는 그대로 유지하는 것이 좋습니다.
이전 예시와 마찬가지로 한 번 실행되거나 두 번 실행하는 것 사이에서 사용자가 볼 수 있는 동작 차이가 없습니다.
실제로 개발 환경에서는 `logVisit`가 아무 작업도 수행하지 않아야 합니다. 개발 환경의 로그가 제품 지표를 왜곡시키지 않도록 하기 위함입니다.
컴포넌트 파일을 저장할 때 마다 재마운트되므로 개발 환경에서는 추가적인 방문 기록을 로그에 남기게 됩니다.

보내지는 분석 이벤트를 디버깅하기 위해서는 앱을 스테이징환경(제품 모드로 실행)에 배포하거나 Strict Mode를 일시적으로 사용 중지하여 개발 환경 전용의 재마운팅 검사를 수행할 수 있습니다.
또한 Effect대신 라우트 변경 이벤트 핸들러에서 분석을 보낼 수 있습니다.
더 정밀한 분석을 위해 Intersection Observer를 사용하여 어떤 컴포넌트가 뷰포트에 있는지 얼마나 오래 보이는지 추적하는데 도움이 될 수 있습니다.

### Effect가 아닌 경우: 애플리케이션 초기화
일부 로직은 애플리케이션 시작 시에 한 번만 실행되어야 합니다. 이러한 로직은 컴포넌트 외부에 배치할 수 있습니다.
```jsx
if (typeof window !== 'undefined') { // 브라우저에서 실행 중인지 확인합니다.
  checkAuthToken();
  loadDataFromLocalStorage();
}

function App() {
  // ...
}
```
컴포넌트 외부에서 해당 로직을 실행하면, 해당 로직은 브라우저가 페이지를 로드한 후 한 번만 실행됨이 보장됩니다.

### Effect가 아닌 경우: 제품 구입하기
가끔은 클린업 함수를 작성하더라도 Effect가 두 번 실행되는 것에 대해 사용자가 확인할 수 있는 결과를 방지할 방법이 없을 수 있습니다.
아래 예시를 보겠습니다.
```js
useEffect(() => {
  // 🔴 잘못된 방법: 이 Effect는 개발 환경에서 두 번 실행되며 코드에 문제가 드러납니다.
  fetch('/api/buy', { method: 'POST' });
}, []);
```
사용자는 제품을 두 번 구매하고 싶지 않을 것입니다. 그러나 이러한 로직은 Effect에 넣지 말아야 하는 이유입니다.
사용자가 다른 페이지로 이동한 다음 뒤로 가기 버튼을 누르는 경우를 생각해보세요.
Effect가 다시 실행됩니다. 사용자가 페이지를 방문할 때 제품을 구매하려고 하지 않으며, 사용자가 구매 버튼을 클릭할 때 제품을 구매하고 싶은 것입니다.

구매는 랜더링에 의해 발생하는 것이 아니라 특정 상호작용에 의해 발생합니다.
Effect가 아닌 Buy 버튼의 이벤트 핸들러로 `/api/buy` 요청을 이동 시켜야 합니다.
```js
  function handleClick() {
    // ✅ 구매는 특정 상호 작용에 의해 발생하는 이벤트입니다.
    fetch('/api/buy', { method: 'POST' });
  }
  ```
  만약 컴포넌트를 다시 마운트 했을 때 애플리케이션의 로직이 깨진다면, 기존에 존재하던 버그가 드러난것입니다.
  사용자의 관점에서 페이지를 방문하는 것과 첫 방문 이후 다른 페이지를 이동했다 되돌아오더라도 차이가 없어야한다는 것입니다.

  