## 따옴표로 문자열 전달하기

어떠한 속성 값을 어트리뷰트에 전달하기 위해선 따옴표로 묶어주어야 함.

```jsx
export default function Avatar() {
  return (
    <img
      className="avatar"
      src="https://i.imgur.com/7vQD0fPs.jpg"
      alt="Gregorio Y. Zara"
    />
  );
}
```

그러나, 속성 값들을 식별자를 통하여 전달할 경우, 자바스크립트모드 {}를 활용 가능

```jsx
export default function Avatar() {
  const avatar = "https://i.imgur.com/7vQD0fPs.jpg";
  const description = "Gregorio Y. Zara";
  return <img className="avatar" src={avatar} alt={description} />;
}
```

## JSX 안에서 중괄호는 두 가지 방법으로만 사용할 수 있음

1. JSX 태그 안의 문자 `<h1>{name}'s To Do List</h1>`
2. = 바로 뒤에 오는 어트리뷰트 `src={avatar}`

JSX에서는 객체도 전달할 수 있는데, 이는 이중 중괄호를 통하여 가능함.

````jsx
    <ul style={{
      backgroundColor: 'black',
      color: 'pink'
    }}>
    ```
````
