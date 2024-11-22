### JSX에서 마크업을 동적으로 조작하고 싶을때 중괄호를 사용해 js를 사용한다.

```jsx
// 문자열(정적) 어트리뷰트는 ',"로 감싸 전달
// 동적인 어트리뷰트는 {}로 전달해 마크업에서 바로 js를 읽게 함
// 객체는 이중 중괄호로 묶어 전달
export default function Avatar() {
  const description = 'Improve the videophone'
  return (
    <ul style={{
      backgroundColor: 'black',
      color: 'pink'
    }}>
      <li className="description-1">{description}</li>
    </ul>
  );
}
```
