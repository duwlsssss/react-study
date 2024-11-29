# Responding to Events
이벤트 핸들러는 클릭, 마우스 호버, 폼 인풋 포커스 등 사용자 상호작용에 따라 유발되는 사용자 정의 함수이다.

## 이벤트 핸들러 추가하기

이벤트 핸들러 추가를 위해서는 먼저 함수를 정의하고 이를 적절한 JSX 태그에 prop 형태로 전달해야 한다. 

```jsx
export default function Button() {
  function handleClick() {  //이벤트 핸들러
    alert('You clicked me!');
  }

  return (
    <button onClick={handleClick}>
      Click me
    </button>
  );
}

```

`handleClick` 함수를 정의하였고 이를 `<button>`에 prop 형태로 전달하였다. 

이벤트 핸들러 함수의 특징 

- 주로 컴포넌트 *내부*에서 정의된다.
- `handle`로 시작하고 그 뒤에 이벤트명을 붙인 함수명을 가진다.

다른 방법으로 이벤트 핸들러를 JSX 내에서 인라인으로 정의할 수 있다.

```jsx
<button onClick={function handleClick() {  alert('You clicked me!');}}>
```

또는 화살표 함수를 사용하여 보다 간결하게 정의할 수도 있다.

```jsx
<button onClick={() => {  alert('You clicked me!');}}>
```


### 이벤트 핸들러에 함수 전달 시 주의사항

1. **함수 전달과 함수 호출의 차이**
    - **함수 전달 (올바름)**
        
        ```jsx
        <button onClick={handleClick}>클릭</button>
        ```
        
        - `handleClick` 함수는 전달만 되고, React가 이벤트 발생 시 호출한다.
    - **함수 호출 (잘못됨)**
        
        ```jsx
        <button onClick={handleClick()}>클릭</button>
        ```
        
        - `handleClick()`은 즉시 실행되며, 이벤트와 관계없이 렌더링 시점에 호출된다. 
2. **인라인 함수 사용 시 주의**
    - **잘못된 예:**
        
        ```jsx
        <button onClick={alert('You clicked me!')}>클릭</button>
        ```
        
        - `alert`가 클릭 이벤트와 상관없이 컴포넌트 렌더링 시 실행된다.
    - **올바른 예:**
        
        ```jsx
        <button onClick={() => alert('You clicked me!')}>클릭</button>
        ```
        
        - 익명 함수로 감싸면, 버튼 클릭 시에만 `alert`가 실행된다.

- 이벤트 핸들러는 **함수 자체를 전달**해야 하며, **함수를 호출하면 안 됨**.
- 인라인으로 작성할 경우, **익명 함수로 감싸야** 의도한 대로 동작함.
- `onClick={handleClick()}`이 아니라 `onClick={handleClick}`이다.

## **이벤트 핸들러 내에서 Prop 읽기**

이벤트 핸들러는 컴포넌트 내부에서 선언되기에 이들은 해당 컴포넌트의 prop에 접근할 수 있다. 

## **이벤트 핸들러를 Prop으로 전달하기**

부모 컴포넌트가 자식의 이벤트 핸들러를 지정하고 싶을 때는, 컴포넌트가 부모로부터 받는 prop을 이벤트 핸들러로 전달한다.

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

위 예제에서 Toolbar 컴포넌트는 PlayButton, UploadButton을 렌더링한다.

- `PlayButton`은 `handlePlayClick`을 `Button` 내 `onClick` prop으로 전달한다.
- `UploadButton`은 `() => alert('Uploading!')`을 `Button` 내 `onClick` prop으로 전달한다.

Button 컴포넌트는 onClick이라는 prop을 받고, 이를 사용해 클릭 시 전달된 함수를 호출하도록 React에 지시한다.



## **이벤트 전파**

이벤트 핸들러는 컴포넌트에 있을 수 있는 모든 하위 컴포넌트의 이벤트도 포착한다. 
이벤트가 트리를 따라 “bubble” 되거나 “전파된다”고 표현한다. 

이때 이벤트는 발생한 지점에서 시작하여 트리를 따라 위로 전달된다.

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

### 이벤트가 전파되는 것을 막으려면

- e.stopPropation()을 호출하면 된다.

 - 이벤트 핸들러는 **이벤트 객체**를 유일 인수로 받는데, 관습적으로 event의 e라고 부른다.

- 이러한 이벤트 오브젝트는 전파를 멈출 수 있게 해준다

 - 따라서 이벤트가 상위 컴포넌트로 전파하지 못하게 하려면 e.stopPropagation을 호출한다.

## **기본 동작 방지**

일부 브라우저 이벤트에는 연결된 기본 동작이 있다. (ex. form의 submit) 이런 기본 동작을 방지하기 위해서는 이벤트 객체에서 e.preventDefault()로 기본 동작의 발생을 막을 수 있다.

```jsx
export default function Signup() {
  return (
    <form onSubmit={e => {
      e.preventDefault();
      alert('Submitting!');
    }}>
      <input />
      <button>Send</button>
    </form>
  );
}
```

- e.stopPropagation은 위 태그에 연결된 이벤트 핸들러의 실행을 중지하고, 

- e.preventDefault는 몇몇 이벤트의 기본 브라우저 동작을 방지하는 역할을 한다.