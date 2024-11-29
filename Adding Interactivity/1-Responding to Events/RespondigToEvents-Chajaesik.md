# React에서는 JSX에 이벤트 핸들러를 추가할 수 있다.
#### 이벤트 핸들러는 클릭, 마우스 호버, 폼 인풋 포커스 등 사용자 상호작용에 따라 유발되는 사용자 정의 함수이다.

 

### 학습 내용
- 이벤트 핸들러를 작성하는 여러가지 방법
- 이벤트 처리 로직을 부모 컴포넌트에서 전달하는 방법
- 이벤트가 전달되는 방식과 이를 멈추는 방법
 

#### 사용자가 버튼을 클릭할 경우 메세지를 보여주는 코드이다.

 
```javascript
export default function Button() {
  function handleClick() {
    alert('You clicked me!');
  }

  return (
    <button onClick={handleClick}> // prop의 형태로 이벤트핸들러를 전달
      Click me
    </button>
  );
}
 ```

#### 위 코드에서 작성한 handleClick과 같은 이벤트 핸들러는 다음의 특징을 갖는다.

- 주로 컴포넌트 내부에서 정의된다
- 함수명이 handle로 시작하고 그 뒤에 이벤트명을 붙인다

### 다양한 이벤트 핸들러 정의 방법

 
```javascript
// JSX 내에서 인라인으로 정의
<button onClick={function handleClick() {
  alert('You clicked me!');
}}>

// 화살표(arrow)함수 사용
<button onClick={() => {
  alert('You clicked me!');
}}>
 ```

### !주의!

#### 이벤트 핸들러로 전달한 함수들은 호출이 아닌 전달이어야 한다.

 

| 함수 전달하기 (올바른 예시)                  | 함수 호출하기 (잘못된 예시)               |
|--------------------------------------|----------------------------------|
| `<button onClick={handleClick}>`   | `<button onClick={handleClick()}>`  |
| `<button onClick={() => alert('...')}>` | `<button onClick={alert('...')}>`   |
 

#### 아래의 코드로 작성하면 , 버튼을 클릭할 때마다 실행이 아닌, 컴포넌트가 렌더링 될 때마다 실행될 것이다.
```javascript
<button onClick={alert('You clicked me!')}>
 ```

#### 만약 이벤트 핸들러를 인라인으로 정의하고자 한다면... 다음과 같이 익명 함수로 감싸야 한다

 
```javascript
<button onClick={() => alert('You clicked me!')}>
 ```

### 이벤트 핸들러 Prop은 원하는대로 정할 수 있다.

 
```javascript
function Button({ onSmash, children }) {
  return (
    <button onClick={onSmash}> // 관습적으로 핸들러는 on으로 시작하여 대문자영문으로 이어진다
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
    </div>
  );
}
 ```

 

### 이벤트 전파

 
- 이벤트 핸들러는 해당 컴포넌트가 어떤 자식 컴포넌트의 이벤트를 수신할 수 있다.

- 이를 이벤트가 트리를 따라 "bubble" 되거나 "전파"된다고 표한한다.

- 이때 이벤트는 발생한 지점에서 시작하여 트리를 따라 위로 전달된다.

 
```javascript
export default function Toolbar() {
  return (
    <div className="Toolbar" onClick={() => {
      alert('You clicked on the toolbar!');
    }}>
      <button onClick={() => alert('Playing!')}>
        Play Movie
      </button>
    </div>
  );
}
 ```

#### 자식 컴포넌트인 button을 누르면 두개의 alert가 실행될 것이고, 

#### 툴바 자체를 클릭한다면 부모의 alert만 실행될 것이다.

 

### 왜냐하면...

 

#### 부여된 JSX태그 내에서만 실행되는 onScroll을 제외한  React 내의 모든 이벤트는 전파되기 때문이다.

 

### 전파 멈추기!

 

#### 이벤트 핸들러는 이벤트 오브젝트를 유일한 매개변수로 받는다.

#### 이 오브젝트(보통 event를 의미하는 e)를 이용하여 다음과 같이 작성하여 전파를 멈춘다`

```javascript
    <button onClick={e => {
      e.stopPropagation(); // 전파 멈춰!
      onClick();
    }}>
```
#### 1. React가 `<button>`에 전달된 onClick 핸들러를 호출합니다.
#### 2. Button에 정의된 해당 핸들러는 다음을 수행합니다.
- `e.stopPropagation()`을 호출하여 이벤트가 더 이상 bubbling 되지 않도록 방지합니다.
- Toolbar 컴포넌트가 전달해 준 onClick 함수를 호출합니다.
#### 3. Toolbar 컴포넌트에서 정의된 위 함수가 버튼의 alert를 표시합니다.
#### 4. 전파가 중단되었으므로 부모인 <div>의 onClick은 실행되지 않습니다.

 

### 기본 동작 방지하기

 

#### submit의 버튼을 클릭하면 페이지 전체가 리로드 되는데, 이것을 방지하려면 다음과 같은 코드를 작성하면 된다.

 
```javascript
    <form onSubmit={e => {
      e.preventDefault(); // 기본 동작 방지!
      alert('Submitting!');
    }}>
 ```

 

### 이벤트 핸들러가 사이드 이펙트를 가질 수 있음에 주의하라.

 

#### 왜냐하면 ...


##### 함수를 렌더링하는 것과 다르게 이벤트 핸들러는 순수할 필요가 없기 때문


