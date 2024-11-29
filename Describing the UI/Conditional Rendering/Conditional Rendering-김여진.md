### 조건부로 jsx 반환하기

```jsx
function Item({ name, isPacked }) {
  let itemContent = name;
  if (isPacked) {
    itemContent = name + " ✅";
  }

  return (
    <li className="item">
      {isPacked ? name + '✅' : name}
    </li>;
    <li className="item">
      {name} {isPacked && '✅'} 
      {/* isPacked가 true면 '✅' 반환, false면 아무것도 반환하지 않음 */}
    </li>;
     <li className="item">
      {itemContent}
    </li>;
  );
}

export default function PackingList() {
  return (
    <section>
      <h1>Sally Ride's Packing List</h1>
      <ul>
        <Item 
          isPacked={true} 
          name="Space suit" 
        />
        <Item 
          isPacked={true} 
          name="Helmet with a golden leaf" 
        />
        <Item 
          isPacked={false} 
          name="Photo of Tam" 
        />
      </ul>
    </section>
  );
}
```

⚠ &&의 왼쪽항이 숫자(0 또는 1)일 경우:
  - 1이면 오른쪽항 반환
  - 0이면 `0` 그대로 반환
  
  React는 falsy한 값을 렌더링하지 않지만, 0은 예외적으로 렌더링함
  그러므로 0일때 오른쪽항을 렌더링하지 않으려면 왼쪽항을 boolean으로 바꿔야 함
  
  ```html
  <li className="item">
      {name} {importance > 0 && `(importance: ${importance})`}
  </li>
  ```