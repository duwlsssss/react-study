아래와 같은 경우 일부 컴포넌트에서는 이부 시스템과 동기화해야 할 수도 있다.

- React의 state를 기준으로 React와 상관없는 구성 요소를 제어
- 서버 연결을 설정
- 구성 요소가 화면에 나타날 때 분석 목적의 로그를 전송
Effect를 사용하면 렌더링 후 특정 코드를 실행하여 React 외부의 시스템과 컴포넌트를 동기화할 수 있다.

 

### 학습 내용
- Effect가 무엇인지
- Effect가 이벤트와 다른 점
- 컴포넌트에서 Effect를 선언하는 법
- 불 필요한 Effect 재실행을 건너뛰는 방법
- 개발 중에 Effect가 두 번 실행되는 이유와 해결 방법
 

### Effect란 무엇이고 이벤트와 어떻게 다른가?
- 렌더링 코드를 주관하는 로직은 컴포넌트의 최상단에 위치하며, props와 state를 적절히 변형해 결과적으로 JSX를 반환한다.( 렌더링 코드는 순수해야한다)
- 이벤트 핸들러는 단순한 계산 용도가 아닌 무언가를 하는 컴폰너트 내부의 중첩 함수이다. 
  - 입력 필드를 업데이트하거나, 제품을 구입하기 위해 HTTP POST 요청을 보내거나
  - 사용자를 다른 화면으로 이동 시킬 수 있다.
  - 이벤트 핸들러에는 특정 사용자 작업(예: 버튼 클릭 또는 입력)으로 발생하는 부수효과('프로그램 상태 변경')을 포함한다.

#### Effect는 렌더링 자체에 의해 발생하는 부수 효과를 특정하는 것으로, 특정 이벤트가 아닌 렌더링에 의해 직저ㅏㅂ 발생한다.

예를 들면  채팅에서 메시지를 보내는 것은 이벤트 이다.

왜냐하면 이것은 사용자가 특정 버튼을 클릭함에 따라 직접적으로 발생한다.

그러나 서버 연결 설정은 Effect이다.

왜냐하면 이것은 컴포넌트의 표시를 주관하는 어떤 상호 작용과도 상관없이 발생해야 한다.,.

Effect는 커밋이 끝난 후에 화면 업데이트가 이루어지고 나서 실행된다.

이 시점이 React 컴포넌트를 외부 시스템(네트워크 또는 써드파티 라이브러리)과 동기화하기 좋은 타이밍이다.

### Effect를 작성하는 법
1. Effect 선언 : 기본적으로 Effect는 모든 DOM commit 이후에 실행된다.
2. Effect 의존성 지정: 대부분의 Effect는 모든 렌더링 후가 아닌 필요할 떄만 다시 실행되야한다.
    - 이것은 의존성을 지정하여 제어할 수 있다.
3. 필요한 경우 클린업 함수 추가
    - 일부 Effect는 작업을 중지, 취소 또는 정리하는데 이것은 클린업 함수(cleanup function)을 반환하여 수행된다.

### 1단계 : Effect 선언하기
컴포넌트 내에서 Effect를 선언하려면, React에서 useEffect 훅을 Import 하세요

`import { useEffect } from 'react';`
그런 다음, 컴포넌트의 최상위 레벨에서 호출하고 Effect 내부에 코드를 넣는다.
```
function MyComponent() {
	useEffect(() => {
    	//이곳의 모든 코드는 *모든* 렌더링 후에 실행된다.
    }
    return <div/>
 }
 ```
useEffect는 화면에 렌더링이 반영될 때까지 코드 실행을 "지연"시킨다.

 

이제 외부 시스템과 동기화하기 위해 어떻게 Effect를 사용할 수 있는지 살펴보자

<VideoPlayer>라는 React 컴포넌트를 살펴보자.

이 컴포넌트를 isPlaying이라는 props를 통해 재생 중인지 일시 정지 상태인지 제어하고자 한다.

<VideoPlayer isPlaying={isPlaying}/>
 

 

커스텀 VideoPlayer 컴포넌트는 내장 브라우저 <video> 태그를 렌더링한다.
```
function VideoPlayer({ src, isPlaying }) {
	// TODO : isPlaying을 활용하여 무언가 수행하기
    return <video src={src}/>
 }
 ```

그러나 <video> 태그에는 isPlaying prop이 존재하지 않는다.

이를 제어하는 유일한 방법은 DOM 요소에서 수동으로 play() 및 pause() 메서드를 호출하는 것이다.

즉 isPlaying pop의 값(현재 비디오가 재생 중인지 여부)을 play() 및 pause()와 같은 호출과 동기화 해야한다.

 

그러기 위해서는 먼저 <video> DOM 노드의 ref를 가져와야 한다.

 

아래는 잘못된 예시이다.
```
  if (isPlaying) {
    ref.current.play();  // 렌더링 중에 이를 호출하는 것이 허용되지 않습니다.
  } else {
    ref.current.pause(); // 역시 이렇게 호출하면 바로 위의 호출과 충돌이 발생합니다.
  }
  ```
그 이유는 렌더링 중에 DOM 노드를 조작하려고 시도하기 때문이다.

React에서는 렌더링이 JSX의 순수한 계산이어야 하며, DOM 수정과 같은 "부수 효과"를 포함해서는 안된다.

게다가, 처음 VideoPlayer가 호출될 떄 해당 DOM이 아직 존재하지 않아 null인 상태이다

React는 컴포넌트가 JSX를 반환할 때 까지 어떤 DOM을 생성할지 모르기 때문에 play() 또는 pause()를 호출할 DOM 노드가 아직 없다.

해결 방법은  부수 효과를 렌더링 연산에서 분리하기 위해 useEffect로 감싸는 것이다.

 
```
  useEffect(() => {
    if (isPlaying) {
      ref.current.play();
    } else {
      ref.current.pause();
    }
  });
 ```

DOM 업데이트를 Effect로 감싸면 React가 화면을 업데이트한 다음에 Effect안의 요소가 실행된다.

 

VideoPlayer 컴포넌트가 렌더링 될 때(처음 호출하거나 다시 렌더링 할 때) 발생하는 일

1. React는 화면을 업데이트하여 <video> 태그가 올바른 속성과 함께 DOM에 있는지 확인
2. React가 Effect를 실행
3. Effect에서는 isPlaying 값에 따라 play() 또는 pause()를 호출할 것이다.
```
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

### !주의! 다음의 코드는 무한 루프를 만들어낸다.
```
const [count, setCount] = useState(0);
useEffect(() => {
  setCount(count + 1);
});
```
Effect는 렌더링의 결과로 실행된다.

state를 설정하면 렌더링이 트리거된다.

즉 Effect가 실행되고 설정되면 재 랜더링이 발생하는 과정이 계속해서 발생한다.

이러한 특성 때문에 Effect는 일반적으로 컴포넌트를 외부시스템과 동기화하는데 사용된다.

외부 시스템이 없고 다른 상태에 기반하여 상태를 조정하려는 경우에는 Effect가 필요하지 않을 수 있다.

### 2단계 : Effect의 의존성 지정하기
우리는 Effect는 모든 렌더링 후에 실행되는 특징을 가진다고 배웠다.

이는 종종 원하는 로직이 아닐 수 있다.

때 때로 느릴 수 있다.
때때로 잘못 될 수 있다.
 

아래 함수는 useEffect의 두번째 인자값으로 "의존성"을 추가하는 코드이다.
```
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
  }, [isPlaying]); //의존성 명시

  return <video ref={ref} src={src} loop playsInline />;
}
 ```

의존성 배열이 없는 경우와 빈 [] 의존성 배열이 있는 경우의 동작이 다르다.
1. useEffect(() => {
  // 모든 렌더링 후에 실행됩니다
});
2. useEffect(() => {
	// 마운트될 때만 실행된다.(컴포넌트가 나타날 때)
},[]);
3. useEffect(() => {
	//마운트될 때 실행되며, 
    // 또한 렌더링 이후 a 또는 b 중 하나라도 변경된 경우에도 실행된다.
},[a,b]);
 

이때 의존성 배열에서는 ref와  useState로 반환되는 set 함수들도 "안정된 식별성(stable identity)"를 가지기 때문에 의존성 배열에서 생략된다.

하지만 예를 들어, ref가 부모 컴포넌트에서 전달되었다면 의존성 배열에 명시해야 한다 

왜냐하면 부모 컴포넌트가 항상 동일한 ref를 전달하는지, 또는 여러  ref 중 하나를 조건부로 전달하는지 알 수 없기 때문이다