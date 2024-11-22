# 조건부 렌더링

React는 if 문, && 및 ? : 연산자와 같은 자바스크립트 문법을 사용하여 조건부로 JSX를 렌더링할 수 있음.

대게 가독성 때문에 if문은 잘 안쓰고 &&, 삼항연산자를 많이 사용하는 것 같음.

가령,

```jsx
if (isPacked) {
  return <li className="item">{name} ✅</li>;
}
return <li className="item">{name}</li>;
```

이 코드 대신

```jsx
return <li className="item">{isPacked ? name + " ✅" : name}</li>;
```

이렇게 쓰거나,

```jsx
  <li className="item">
    {name} {isPacked && '✅'}
  </li>
);
```

이렇게 쓸 수 있음.

&&연산자가 둘 중 하나라도 거짓이면 false를 반환하기에, isPacked 포지션에 true/ false 를 두고 뒤에 false가 되는 값6개가 아닌 다른 것을 써서 사용함. 문서에 나와있는 것 처럼 모달형식 컴포넌트에 주로 사용하는 것 같음.

### 챌린지 답안 비교

```jsx
function Item({ name, importance }) {
  return (
    <li className="item">
      {name} {importance !== 0 ? `(importance:${importance})` : ""}
    </li>
  );
}
```

내가 작성한 코드. 삼항 연산자와 템플릿 리터럴 활용

```jsx
function Item({ name, importance }) {
  return (
    <li className="item">
      {name}
      {importance > 0 && " "}
      {importance > 0 && <i>(Importance: {importance})</i>}
    </li>
  );
}
```

답안코드 논리연산자 활용
