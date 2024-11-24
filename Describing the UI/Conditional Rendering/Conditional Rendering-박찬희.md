# Conditional Rendering

컴포넌트는 조건에 따라 다른 항목을 표시해야 하는 경우가 많습니다. React는 `if`문, `&&` 및 `?  :` 연산자와 같은 자바스크립트 문법을 조건부로 JSX를 랜더링할 수 있습니다.

## 조건부로 JSX 반환하기
짐을 챙겼는지 안 챙겼는지 표시할 수 있는 여러 개의 `Item`을 렌더링하는 `PackingList` 컴포넌트가 있다고 가정해봅시다.
```jsx
function Item({ name, isPacked }) {
  return <li className="item">{name}</li>;
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
`Item `컴포넌트 중 일부는 `isPacked` prop이 `false`가 아닌 `true`로 설정되어 있습니다. `isPacked={true}`인 경우 짐을 챙긴 항목에 체크 표시(✅)를 추가하려고 합니다.

다음과 같이 if/else 문으로 작성할 수 있습니다.
```jsx
if (isPacked) {
  return <li className="item">{name} ✅</li>;
}
return <li className="item">{name}</li>;
```

`isPacked `prop이 `true`이면 이 코드는 다른 JSX 트리를 반환합니다. 이로 인해 일부 항목은 끝에 체크 표시가 있습니다.

```jsx
function Item({ name, isPacked }) {
  if (isPacked) {
    return <li className="item">{name} ✅</li>;
  }
  return <li className="item">{name}</li>;
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

JavaScript의 `if`와 `return`문으로 분기 로직을 만드는 방법을 살펴보세요. React에서 제어흐름(예:조건문)은 JavaScript로 처리합니다.

## 조건부로 null을 사용하여 아무것도 반환하지 않기
어떤 경우에는 아무것도 렌더링하고 싶지 않을 수 있습니다. 컴포넌트는 반드시 무언가를 반환해야 하는데 이 경우에 `null`을 반환할 수 있습니다. 다음과 같이 말이죠.

```js
if (isPacked) {
  return null;
}
return <li className="item">{name}</li>;
```
`isPacked`가 `true`라면 컴포넌트는 아무것도 반환하지 않지만, `false`라면 JSX가 반환될 것입니다.

```js
// 짐을 챙겼다면 null을 반환해 챙기지 않은 항목만 랜더링
function Item({ name, isPacked }) {
  if (isPacked) {
    return null;
  }
  return <li className="item">{name}</li>;
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

실제로 컴포넌트에서 `null`을 반환하는 것은 개발자가 렌더링하려고 할 때 놀랄 수 있기 때문에 흔한 경우는 아닙니다. 더 자주, 부모 컴포넌트 JSX에 컴포넌트를 조건부로 포함하거나 제외할 수 있습니다. 

## 조건부로 JSX 포함시키기

이전 예시에서는 어떤 항목(있는 경우)을 제어했습니다. 컴포넌트에 의해 JSX 트리가 반환되었습니다. 렌더링된 출력 결과에서 이미 일부 중복이 발견되었을 수 있습니다.

```js
//두 조건부 분기가 모두 
//<li className="item">...</li>를 반환합니다.
if (isPacked) {
  return <li className="item">{name} ✅</li>;
}
return <li className="item">{name}</li>;
```

이 중복코드가 나쁜 건 아니지만, 코드를 유지 보수하기 어렵게 만들 수 있습니다. 예를 들어 className을 바꾸고 싶다면 코드상 두 곳을 수정해야 합니다. 이러한 상황에서 조건부로 약간의 JSX를 포함해 코드를 더 DRY(덜 반복적이게)하게 만들 수 있습니다.

### 삼항 조건 연산자(? :)
JavaScript는 조건 연산자 또는 “삼항 조건 연산자”라는 조건식을 작성하기 위한 간단한 문법이 있습니다.

위 예시 코드 대신 다음과 같이 작성할 수 있습니다.
```js
return (
  <li className="item">
    {isPacked ? name + ' ✅' : name}
  </li>
);
```
”`isPacked`가 참이면 (`?`) `name + ' ✔'`을 렌더링하고, 그렇지 않으면 (`:`) `name`을 렌더링한다.” 라고 읽을 수 있습니다.

이제 완성된 항목의 텍스트를 `<del>`과 같은 다른 HTML 태그로 줄 바꿈 하여 삭제하려고 합니다. 더 많은 JSX를 중첩하기 쉽도록 새로운 줄과 괄호를 추가할 수 있습니다.
```jsx
function Item({ name, isPacked }) {
  return (
    <li className="item">
      {isPacked ? (
        <del>
          {name + ' ✅'}
        </del>
      ) : (
        name
      )}
    </li>
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

이 스타일은 간단한 조건에 잘 어울리지만, 적당히 사용하는 게 좋습니다. 중첩된 조건부 마크업이 너무 많아 컴포넌트가 지저분해질 경우 자식 컴포넌트를 추출하여 정리하세요. React에서 마크업은 코드의 일부이므로 변수 및 함수와 같은 도구를 사용하여 복잡한 식을 정리할 수 있습니다.

## 논리 AND 연산자 (`&&`)
또 다른 일반적인 손쉬운 방법은 JavaScript 논리 AND (’&&’) 연산자입니다. React 컴포넌트에서는 조건이 참일 때 일부 JSX를 렌더링하거나 그렇지 않으면 아무것도 렌더링하지 않을 때 를 나타내는 경우가 많습니다. 다음과 같이 `&&`를 사용하면 `isPacked`가 `true`인 경우에만 조건부로 체크 표시를 렌더링할 수 있습니다.
```jsx
return (
  <li className="item">
    {name} {isPacked && '✅'}
  </li>
);
```
이것을 “`isPacked`이면 (`&&`) 체크 표시를 렌더링하고, 그렇지 않으면 아무것도 렌더링하지 않습니다.”라고 읽을 수 있습니다.

논리연산자를 활용하여 코드를 수정하면 아래와 같습니다.
```jsx
function Item({ name, isPacked }) {
  return (
    <li className="item">
      {name} {isPacked && '✅'}
    </li>
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
JavaScript && 표현식은 왼쪽(조건)이 `true`이면 오른쪽(체크 표시)의 값을 반환합니다. 그러나 조건이 `false`이면 전체 표현 식이 `false`가 됩니다. React는 `false`를 `null` 또는 `undefined`처럼 JSX 트리의 “구멍”으로 간주하고 그 자리에 아무것도 렌더링하지 않습니다.

### 주의!
`&&`의 왼쪽에 숫자를 두지 마세요.

조건을 테스트하기 위해 JavaScript는 자동으로 왼쪽을 `boolean`으로 변환합니다. 그러나 왼쪽이 `0`이면 전체 식이 (`0`)을 얻게 되고, React는 아무것도 아닌 `0`을 렌더링할 것입니다.

예를 들어, 흔하게 하는 실수로 `messageCount && <p>New messages</p>`와 같은 코드를 작성하는 것입니다. 메시지 카운트가 `0`일 때 아무것도 렌더링하지 않는다고 쉽게 추측할 수 있지만, 실제로는 `0` 자체를 렌더링합니다!

이 문제를 해결하려면 `messageCount > 0 && <p>New messages</p>` 처럼 왼쪽을 `boolean`으로 만드세요.

## 변수에 조건부로 JSX를 할당하기
위와 같은 방법이 일반 코드를 작성하는 데 방해가 되면 `if` 문과 변수를 사용하세요. `let`으로 정의된 변수는 재할당할 수 있으므로 표시할 기본 내용인 이름을 먼저 대입하세요.
```jsx
let itemContent = name;
```
`if` 문을 사용하여 `isPacked`가 `true`인 경우 JSX 표현식을 `itemContent`에 다시 할당합니다.
```jsx
if (isPacked) {
  itemContent = name + " ✅";
}
```
반환된 JSX 트리에 중괄호를 사용하고 이전에 계산된 식을 JSX 내부에 중첩하여 변수를 포함합니다.
```jsx
<li className="item">
  {itemContent}
</li>
```
이 스타일은 가장 유연합니다. 다음과 같이 수정할 수 있습니다.
```jsx
function Item({ name, isPacked }) {
  let itemContent = name;
  if (isPacked) {
    itemContent = name + " ✅";
  }
  //   if (isPacked) {
  //   itemContent = (
  //     <del>
  //       {name + " ✅"}
  //     </del>
  //   );
  // }
  // 주석 처리된 코드를 활용하여 텍스트 뿐만 아니라 임의의 JSX에도 문제 없이 작동하는 것을 알 수 있습니다.
  return (
    <li className="item">
      {itemContent}
    </li>
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
