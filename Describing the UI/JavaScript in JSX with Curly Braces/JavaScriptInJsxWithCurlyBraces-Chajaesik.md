## JSX에서 동적인 프로퍼티 참조

마크업 내부에서 동적인 프로퍼티를 참조하고 싶을 때 JSX에서 중괄호를 사용하여 JS를 사용할 수 있다.

### 학습 내용
- 따옴표로 문자열 전달하는 방법
- 중괄호가 있는 JSX 안에서 JS 변수를 참조하는 방법
- 중괄호가 있는 JSX 안에서 JS 함수를 호출하는 방법
- 중괄호가 있는 JSX 안에서 JS 객체를 사용하는 방법

### 문자열 어트리뷰트를 JSX에 전달하는 방법
문자열 어트리뷰트를 JSX에 전달하려면 작은 따옴표나 큰 따옴표로 묶어야 한다.

```jsx
<img
  className="avatar"
  src="https://i.imgur.com/7vQD0fPs.jpg"
  alt="Gregorio Y. Zara"
/>
```

### JS 변수를 참조하는 방법
`src`나 `alt`를 동적으로... 즉 JS 변수를 참조하려면 큰따옴표(`"`)를 중괄호(`{`)로 바꿔 JS의 값을 사용한다.

```javascript
const avatar = 'https://i.imgur.com/7vQD0fPs.jpg';

return (
  <img
    className="avatar"
    src={avatar}
  />
);
```

### JS 함수 호출하는 방법
아래 코드에서 사용한 `formatDate()`와 같은 함수 호출을 포함해 모든 JS 표현식은 중괄호 사이에서 작동한다.

```javascript
const today = new Date();

function formatDate(date) {
  return new Intl.DateTimeFormat(
    'ko-KR',
    { weekday: 'long' }
  ).format(date);
}

export default function TodoList() {
  return (
    <h1>To Do List for {formatDate(today)}</h1> // JSX 안에서 함수 호출
  );
}
```

### JS 객체를 사용하는 방법
JSX 중괄호 안에서 객체를 사용할 수 있다.

```javascript
const person = { // 객체 선언
  name: 'Gregorio Y. Zara',
  theme: {
    backgroundColor: 'black',
    color: 'pink'
  }
};

export default function TodoList() {
  return (
    <div style={person.theme}>
      <h1>{person.name}'s Todos</h1> // 중괄호로 참조
    </div>
  );
}
```

### JSX 안에서 중괄호 사용
JSX 안에서 중괄호는 두 가지 방법으로만 사용할 수 있다.

1. **JSX 태그 안의 문자**:
   ```jsx
   <h1>{name}'s To Do List</h1> // 작동
   <{tag}>Gregorio Y. Zara's To Do List</{tag}> // 오작동
   ```

2. **= 바로 뒤에 오는 어트리뷰트**:
   ```jsx
   src={avatar} // avatar 변수를 읽지만 
   src="{avatar}" // "{avatar}" 문자열을 전달
   ```