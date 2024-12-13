# useEffect란?
- useEffect는 React의 Hook으로, 컴포넌트가 렌더링된 이후 특정 작업을 수행하기 위해 사용됩니다. 
- API 호출
- DOM 조작
- 타이머 설정
- 이벤트 구독 등

## 렌더링과 부수 효과의 차이
렌더링
- React 컴포넌트의 UI를 계산하는 로직.
- 순수해야 하며 DOM 변경 같은 부수 효과를 포함해서는 안 됩니다.

부수 효과

- 화면이 렌더링된 후 발생해야 하는 작업.
- 외부 API 호출, DOM 노드 접근, 서버 연결 등의 작업.

## useEffect의 기본 사용법

1. useEffect 선언
``` javascript

import { useEffect } from 'react';

function MyComponent() {
useEffect(() => {
console.log('Effect 실행');
});

return <div>My Component</div>;
}
```

- 컴포넌트가 렌더링된 후, useEffect 내부의 코드를 실행합니다.
- React는 화면을 업데이트한 후 useEffect를 실행하므로 DOM 조작에 적합합니다.
## useEffect로 DOM 조작 예제
비디오 재생 제어
``` javascript

import { useRef, useEffect } from 'react';

function VideoPlayer({ isPlaying }) {
const videoRef = useRef(null);

useEffect(() => {
if (isPlaying) {
videoRef.current.play();
} else {
videoRef.current.pause();
}
}, [isPlaying]); // isPlaying이 변경될 때만 실행

return <video ref={videoRef} src="video.mp4" />;
}
```

- isPlaying 값에 따라 play() 또는 pause() 메서드를 호출.
- [] 의존성 배열: 특정 값이 변경될 때만 useEffect를 실행.

## 클린업 함수
왜 클린업이 필요한가?
- 컴포넌트가 사라지거나(언마운트) 다시 렌더링될 때, 이전 작업을 정리하지 않으면 메모리 누수나 이전 상태 유지 문제가 발생

클린업 사용법
``` javascript

useEffect(() => {
const timerId = setInterval(() => {
console.log('Interval 실행');
}, 1000);

return () => {
clearInterval(timerId); // 이전 타이머 정리
};
}, []);
```

## React의 렌더링 흐름과 클린업
1. 컴포넌트 렌더링 → Effect 실행.
2. 새 렌더링 발생
    - 이전 Effect의 클린업 실행.
    - 새 Effect 실행.
3. 컴포넌트 언마운트
    - 최종 클린업 실행.
## useEffect의 의존성 배열
의존성 배열이란? useEffect가 언제 실행될지 제어

### 의존성 배열 
- 없음 =>  (useEffect(() => {...})) 모든 렌더링 후 실행

- 빈 배열 ([]) => 컴포넌트가 마운트될 때 한 번만 실행

- 특정 값 포함 ([dep]) => 해당 값(dep)이 변경될 때만 실행


``` javascript

useEffect(() => {
console.log('Effect 실행');
}, [value]); // value가 변경될 때만 실행
```

### 개발 모드에서 Effect 두 번 실행되는 이유
React는 개발 모드에서 Effect를 두 번 실행하여 클린업 함수가 올바르게 구현되었는지 확인합니다. 배포 모드에서는 한 번만 실행됩니다.

``` javascript
useEffect(() => {
console.log('Effect 실행');
return () => console.log('Effect 정리');
}, []);
```
- 개발 모드에서는 Effect 실행 → Effect 정리 → Effect 실행 순서로 실행
- 배포 모드에서는 Effect 실행만 실행

## 주의할 점
#### 1. Effect에서 상태 업데이트로 무한 루프 주의:

``` javascript

useEffect(() => {
setCount(count + 1); // ❌ 무한 루프 발생
});
```
#### 2. 의존성 배열의 값 정확히 명시:

- 모든 의존성을 포함해야 React가 올바르게 실행을 제어.
#### 3. Effect 내부에서 DOM 조작 피하기:

- 렌더링 중 DOM 조작은 React의 흐름과 충돌합니다.
#### 4. Effect 대신 이벤트 핸들러 사용:

특정 작업은 사용자 상호작용 이벤트에 위치해야 함.
``` javascript

function handleClick() {
fetch('/api/buy', { method: 'POST' }); // 이벤트 기반 처리
}
```
