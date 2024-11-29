### 1. jsx에 이벤트 핸들러 추가하기

함수를 정의하고 특정 JSX 태그에 prop 형태로 전달

```jsx
const Button = ({ text, ...rest }) => { 
  // onClick prop을 받아 브라우저 빌트인 <button>의 onClick={onClick}으로 전달함
  return (
    <StyledButton {...rest}>
      {text}
    </StyledButton>
  )
}
// 이벤트 핸들러를 Button에 prop으로 전달함
const ToDoList = () => {
  ...
  return (
    <StyledContainer>
      <div className="btn-container">
      <Button
        text="수정"
        onClick={() => {
          setEditId(todo.id);
          todoEdit.setValues({ title: todo.title, content: todo.content });
        }}
      />
      <Button text="상세보기" onClick={() => navigate(`/todo/${todo.id}`)}/> {/*익명 함수로 전달*/}
      <Button text="삭제" onClick={() =>{deleteTodo.mutate(todo.id); alert("삭제되었습니다!");}}/>
    </div>
    </StyledContainer>
  )
}
```

#### ⚠ 이벤트 핸들러는 추후의 이벤트에 의해 호출되도록 **전달**하는 것임

```jsx
<button onClick={handleClick}>
<button onClick={() => alert('...')}>

{/* JSX의 {} 내의 자바스크립트는 즉시 실행되므로 함수 호출을 넣으면 클릭이 없어도 렌더링될때마다 실행됨 */}
<button onClick={handleClick()}>
<button onClick={alert('...')}>
```

<br/>

### 2. 이벤트 전파 방지

이벤트는 트리를 따라 발생한 지점에서 위로 전파되며(bubbleing) 특정 컴포넌트가 자식 컴포넌트의 이벤트를 수신할 수도 있음

```jsx
// Play Movie, Upload Image 버튼 클릭시 부모 컴포넌트의 alert('You clicked on the toolbar!')도 실행됨
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

// 이벤트 핸들러는 이벤트 오브젝트(event)를 유일한 매개변수로 받음
// 이 오브젝트를 이벤트의 정보를 읽어들이는데 사용함 
// 이 오브젝트로 이벤트 전파를 멈출 수도 있음
function Button({ onClick, children }) {
  return (
    <button onClick={e => {
      e.stopPropagation(); {/*e.stopPropagation()을 호출해 이벤트가 더 이상 bubbling되지 않도록(상위 태그에서 실행되지 않도록) 방지*/}
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

### 3. 브라우저 기본 동작 방지

일부 브라우저 이벤트는 기본 동작을 가짐
<form>의 제출 이벤트는 그 내부의 버튼 클릭시 페이지 전체를 리로드함

```jsx
export default function Signup() {
  return (
    <form onSubmit={e => {
      e.preventDefault(); {/*e.preventDefault()로 기본 브라우저 동작을 가진 이벤트가 그 동작을 실행하지 않도록 함*/}
      alert('Submitting!');
    }}>
      <input />
      <button>Send</button>
    </form>
  );
}
```

#### 이벤트 핸들러는 순수하지 않아도됨. 즉 무언가를 변경할 수 있음



