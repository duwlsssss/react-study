# 이벤트에 응답하기

이벤트 핸들러란?
클릭, 마우스 호버, 폼 인풋 포커스 등 사용자 상호작용에 따라 유발되는 사용자 정의 '함수' 입니다.

```jsx
export default function Button() {
  function handleClick() {
    alert('You clicked me!');
  }

  return (
    <button onClick={handleClick}> //<button>에 prop 형태로 전달
      Click me
    </button>
  );
}
```

이렇게 사용하거나 또는 인라인 방식이나 화살표 함수로도 사용이 가능

```jsx
<button onClick={function handleClick() {
alert('You clicked me!');
}}>

<button onClick={() => {
alert('You clicked me!');
}}>
```

### 왜 props형태로 전달하나? => React의 이벤트 처리 메커니즘과 함수 호출의 타이밍과 관련이 있음

1. {handleClick}: 참조만 전달하여 react가 함수 실행을 컨트롤 할 수 있도록함
2. {handleClick()} : 이벤트 핸들러에 함수를 즉시 실행한 결과 반환 => 이벤트 발생전에 이미 함수를 실행

래퍼를 사용하여 onClick={() => handleClick()} 이렇게도 사용 가능하나, 매개변수가 없는 불필요한 래퍼 사용은 성능을 위해 지양하는 것을 권장

## 이벤트 핸들러 내에서 Prop 읽기

이벤트 핸들러는 컴포넌트 내부에서 선언되기에 이들은 해당 컴포넌트의 prop에 접근 가능

```jsx
function AlertButton({ message, children }) {
  return <button onClick={() => alert(message)}>{children}</button>;
}

export default function Toolbar() {
  return (
    <div>
      <AlertButton message="Playing!">Play Movie</AlertButton>
      <AlertButton message="Uploading!">Upload Image</AlertButton>
    </div>
  );
}
```

## 이벤트 전파 ( Event propagation )

부모-자식간의 관계로 인해 나오는 동작

```jsx
export default function Toolbar() {
  return (
    <div
      className="Toolbar"
      onClick={() => {
        alert("You clicked on the toolbar!");
      }}
    >
      <button onClick={() => alert("Playing!")}>Play Movie</button>
      <button onClick={() => alert("Uploading!")}>Upload Image</button>
    </div>
  );
}
```

이걸 막기 위해, `e.stopPropagation(); `사용이 가능

```jsx
function Button({ onClick, children }) {
  return (
    <button
      onClick={(e) => {
        e.stopPropagation(); //이거!
        onClick();
      }}
    >
      {children}
    </button>
  );
}

export default function Toolbar() {
  return (
    <div
      className="Toolbar"
      onClick={() => {
        alert("You clicked on the toolbar!");
      }}
    >
      <Button onClick={() => alert("Playing!")}>Play Movie</Button>
      <Button onClick={() => alert("Uploading!")}>Upload Image</Button>
    </div>
  );
}
```

## 브라우저 리로드 방지

<form>의 제출 이벤트는 그 내부의 버튼을 클릭 시 페이지 전체를 리로드함.
이를 막기위해  e.preventDefault() 사용 가능

```jsx
export default function Signup() {
  return (
    <form
      onSubmit={(e) => {
        e.preventDefault(); //이거
        alert("Submitting!");
      }}
    >
      <input />
      <button>Send</button>
    </form>
  );
}
```

### 함수를 렌더링하는 것과 다르게 이벤트 핸들러는 순수할 필요가 없음
