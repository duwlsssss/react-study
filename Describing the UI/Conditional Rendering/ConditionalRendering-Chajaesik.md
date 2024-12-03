
# React에서 조건부 랜더링

컴포넌트는 조건에 따라 다른 항목을 표시해야 하는 경우가 많은데, React는 `if`문, `&&` 및 `?:` 연산자와 같은 자바스크립트 문법을 사용하여 조건부로 JSX를 랜더링 할 수 있다.

## `if`문을 활용한 조건부 랜더링

```javascript
function Item({ name, isPacked }) { // isPacked는 true 또는 false의 값을 갖는다.
  if (isPacked) {
    return <li className="item">{name} ✅</li>;
  }
  return <li className="item">{name}</li>; 
}

export default function PackingList() {
  return (
    <section>
      <ul>
        <Item 
          isPacked={true} 
          name="Space suit" 
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


### 조건부로 `null`를 사용하면 아무것도 반환하지 않는다.

주로 쓰이진 않는데, 그 이유는 '의도적'으로 값을 반환하는지 다른 사람이 모를 수 있기 때문

```javascript
if (isPacked) {
  return null;
}
```

### 삼항 조건 연산자 (? : )


구조는 `조건 ? true인경우 : false의 경우` 로 작성된다. 아래 코드 예시를 보자


```javascript
return (
  <li className="item">
    {isPacked ? name + ' ✅ ' : name} // 삼항 조건 연산자 사용
        조건             참     거짓
    {isPacked ? name + ' ✅ '} // 거짓인 경우 아무것도 수행하지 않는다.
  </li>
);
```

### 논리 AND 연산자 (&&)


조건이 **참**인 경우에만 렌더링, 
```javascript
return (
  <li className="item">
    {name} {isPacked && '✅'} isPacked가 true인 경우 렌더링
  </li>
);
```
### !작성 방식!

#### 조건이 &&의 왼쪽에 위치해야하고 숫자 사용에 주의하라

왜냐하면 보통의 count값은 0으로 설정하는데, 이를 JS에서 거짓으로 판단하기 때문이다.

```javascript
messageCount && <p>New messages</p> // 잘못된 코드 예시
messageCount > 0 && <p>New messages</p> // 참 잘했어요
```