# Responding to Events

## 이벤트 핸들러 추가하기

이벤트 핸들러 추가를 위해 함수를 정의하고 JSX 태그에 prop 형태로 전달해야 합니다.

#### 이벤트 핸들러 추가 과정
1. 컴포넌트 내부에 함수를 선언
2. 해당 함수 로직 구현
3. JSX에 prop으로 전달
```jsx
export default function Button() {
  function handleClick() {
    alert('You clicked me!');
  }

  return (
    <button onClick={handleClick}>
      Click me
    </button>
  );
}
```
위 예시는 `handleClick` 함수를 정의하여 `<button>`에 prop형태로 전달했습니다. 여기서 `handleClick`은 이벤트 핸들러로 이벤트 핸들러는 다음과 같은 특징을 가집니다.

- 주로 컴포넌트 내부에서 정의
- `handle`로 시작하고 그 뒤에 이벤트 명을 붙인 함수명을 가짐(컨벤션)

#### 이벤트 핸들러 인라인 정의
 이벤트 핸들러는 JSX 내에서 인라인으로 정의할 수 있습니다.
```jsx
<button onClick={function handleClick() {
  alert('You clicked me!');
}} />
// 아래는 화살표 함수 두 방법 모두 동일한 결과
<button onClick={() => {
  alert('You clicked me!');
}}>
```

> **주의!** <br>
이벤트 핸들러로 전달한 함수는 호출이 아닌 전달되어야 합니다. <br>
`handleClick()` 이런 식으로 호출할 경우 랜더링 과정 중 클릭이 없음에도 즉시 함수를 실행<br>
인라인으로 코드 작성시 익명의 함수로 감싸줘야 합니다.<br>
ex `<button onClick={()=> alert('...')}>`

## 이벤트 핸들러 내에서 Prop 읽기

이벤트 핸들러는 컴포넌트 내부에서 선언되기 때문에 해당 컴포넌트의 prop에 접근할 수 있습니다.

```jsx
function AlertButton({ message, children }) {
  return (
    <button onClick={() => alert(message)}>
      {children}
    </button>
  );
}

export default function Toolbar() {
  return (
    <div>
      <AlertButton message="Playing!">
        Play Movie
      </AlertButton>
      <AlertButton message="Uploading!">
        Upload Image
      </AlertButton>
    </div>
  );
}
```
위 예제의 두 버튼에서는 서로 다른 메시지를 표시할 수 있습니다.

## 이벤트 핸들러를 Prop으로 전달하기
이벤트 핸들러는 부모 컴포넌트로 Prop을 받아 자식의 이벤트 핸들러로 전달 할 수 있습니다.
```jsx
function Button({ onClick, children }) {
  return (
    <button onClick={onClick}>
      {children}
    </button>
  );
}

function PlayButton({ movieName }) {
  function handlePlayClick() {
    alert(`Playing ${movieName}!`);
  }

  return (
    <Button onClick={handlePlayClick}>
      Play "{movieName}"
    </Button>
  );
}

function UploadButton() {
  return (
    <Button onClick={() => alert('Uploading!')}>
      Upload Image
    </Button>
  );
}

export default function Toolbar() {
  return (
    <div>
      <PlayButton movieName="Kiki's Delivery Service" />
      <UploadButton />
    </div>
  );
}
```

위 코드는 `Toolbar` 컴포넌트가 `PlayButton`과 `UploadButton`을 랜더링 합니다.
- `PlayButton`은 `handlePlayClick`을 `Button`내 `onClick` prop으로 전달
- `UploadButton`은 `() => alert('Uploading!')`을 `Button`내 `onClick` prop으로 전달
- `Button` 컴포넌트는 `onClick` prop을 받아 `<button>`의 `onClick={onclick}`으로 직접 전달, 클릭시 호출

만약 디자인 시스템을 적용한다면 버튼과 같은 컴포넌트는 동작을 지정하지 않고 스타일만 지정하는 것이 일반적입니다.

## 이벤트 핸들러 Prop 명명하기

사용자 정의 컴포넌트는 이벤트 핸들러 prop의 이름을 원하는 대로 명명할 수 있습니다.
관습적으로 prop의 이름을 `on`으로 시작한 대문자 영문으로 만듭니다.
```jsx
function Button({ onSmash, children }) {
  return (
    <button onClick={onSmash}>
      {children}
    </button>
  );
}

export default function App() {
  return (
    <div>
      <Button onSmash={() => alert('Playing!')}>
        Play Movie
      </Button>
      <Button onSmash={() => alert('Uploading!')}>
        Upload Image
      </Button>
    </div>
  );
}
```
`<Button onSmash={() => alert('Playing!')}>` 이 처럼 직접 정의한 prop을 전달 할 수 있지만 `<button onClick={onSmash}>` 여전히 `onClick` prop을 필요로 합니다.<br>
사용자 컴포넌트의 여부를 제대로 인지하고 prop을 정의해야 합니다. 또한 정의 할때 `onPlayMovie`, `onUploadImage` 등 해당 prop로 무엇을 할 것인지에 대해 명명한다면 활용성이 좋아집니다.


## 이벤트 전파
이벤트 핸들러는 해당 컴포넌트의 자식 컴포넌트의 이벤트를 수신할 수도 있습니다.
이벤트는 발생한 지점에서 시작하여 트리를 따라 위로 전달됩니다.
```jsx
export default function Toolbar() {
  return (
    <div className="Toolbar" onClick={() => {
      alert('You clicked on the toolbar!');
    }}>
      <button onClick={() => alert('Playing!')}>
        Play Movie
      </button>
      <button onClick={() => alert('Uploading!')}>
        Upload Image
      </button>
    </div>
  );
}
```
위 예제의 두 버튼을 클릭하면 버튼의 `onClick`이 먼저 실행되고 그 이후 `<div>`의 `onClick`이 실행되며 2개의 메시지가 표시 됩니다. 또한 툴바만 클릭했을 경우에는 `<div>`의 `onClick` 만 실행됩니다.

>**주의!** <br>
`onScroll`은 부여된 JSX 태그 내에서만 실행되어 전파되지 않습니다.

## 전파 멈추기

이벤트 핸들러는 이벤트 오브젝트를 유일한 매개변수로 받습니다. "event"의 `e`로 호출 되는 것이 일반적으로 이벤트 정보를 읽어내는데 사용합니다.

이벤트가 부모 컴포넌트에 닿지 못하도록 막기 위해서는 `e.stopPropagation()`를 호출하면 됩니다.

```jsx
function Button({ onClick, children }) {
  return (
    <button onClick={e => {
      e.stopPropagation();
      onClick();
    }}>
      {children}
    </button>
  );
}

export default function Toolbar() {
  return (
    <div className="Toolbar" onClick={() => {
      alert('You clicked on the toolbar!');
    }}>
      <Button onClick={() => alert('Playing!')}>
        Play Movie
      </Button>
      <Button onClick={() => alert('Uploading!')}>
        Upload Image
      </Button>
    </div>
  );
}
```
위 예시에 `e.stopPropagation()`를 통해 `Button` 컴포넌트에 포함되어 있는 `<button>`을 클릭했을 때 부모 컴포넌트에 있는 `onClick={() => alert('...')`이 실행되지 않도록 했습니다.

## 기본 동작 방지하기

`<form>`의 제출 이벤트같이 기본적인 내부 이벤트를 방지 하기 위해서는 `e.preventDefault()`를 호출합니다.
- `e.stopPropagation()`은 이벤트 핸들러가 상위 태그에 실행되지 않도록 멈춥니다.
- `e.preventDefault()`는 기본 동작을 싱행하지 않도록 방지합니다.

### 이벤트 핸들러는 사이드 이펙트를 가질 수 있다!
이벤트 핸들러는 순수할 필요가 없기 때문입니다.