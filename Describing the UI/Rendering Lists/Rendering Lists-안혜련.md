# Rendering Lists
데이터 모음에서 유사한 컴포넌트를 여러 개 표시하고 싶을 때가 종종 있다. 

JavaScript 배열 메서드를 사용하여 데이터 배열을 조작할 수 있습니다. 이 페이지에서는 React에서 [`filter()`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Array/filter)와 [`map()`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Array/map)을 사용해 데이터 배열을 필터링하고 컴포넌트 배열로 변환해보자~~

## 배열을 데이터로 렌더링 하기

React에서 filter와 map을 이용해 데이터 배열을 필터링하고 컴포넌트 배열로 반환할 수 있다.

```jsx
<ul>
  <li>Creola Katherine Johnson: mathematician</li>
  <li>Mario José Molina-Pasquel Henríquez: chemist</li>
  <li>Mohammad Abdus Salam: physicist</li>
  <li>Percy Lavon Julian: chemist</li>
  <li>Subrahmanyan Chandrasekhar: astrophysicist</li>
</ul>
```

이런 콘텐츠 목록이 있을 때, 해당 데이터에 뭔가를 추가하게 되면 수동으로 코드를 고쳐줘야 한다. 

이 때 해당 데이터를 구조화 하면 유지보수에도 용이하고 코드의 가독성도 좋아진다.

```jsx
const people = [
  'Creola Katherine Johnson: mathematician',
  'Mario José Molina-Pasquel Henríquez: chemist',
  'Mohammad Abdus Salam: physicist',
  'Percy Lavon Julian: chemist',
  'Subrahmanyan Chandrasekhar: astrophysicist'
];

export default function List() {
  const listItems = people.map(person =>
    <li>{person}</li>
  );
  return <ul>{listItems}</ul>;
}
```

이런 식으로 배열에 문자열을 넣어주고 map으로 순회해 보여줄 수 있다. (물론 이 때 목록의 각 자식에는 고유한 key prop이 필요하다는 에러가 뜬다.)

이 데이터를 좀 더 구조화 하면 아래와 같이 바꿀 수 있다.

## 배열의 항목들을 필터링하기

```jsx
const people = [{
  id: 0,
  name: 'Creola Katherine Johnson',
  profession: 'mathematician',
}, {
  id: 1,
  name: 'Mario José Molina-Pasquel Henríquez',
  profession: 'chemist',
}, {
  id: 2,
  name: 'Mohammad Abdus Salam',
  profession: 'physicist',
}, {
  name: 'Percy Lavon Julian',
  profession: 'chemist',
}, {
  name: 'Subrahmanyan Chandrasekhar',
  profession: 'astrophysicist',
}];
```

여기서 profession: 'chemist'인 사람만 추출하고 프로필 사진을 추가해 화면에 보여주려고 한다. 

chemist인 사람만 찾아내 배열로 반환하는 filter와 해당 배열을 순회하는 map으로 구현할 수 있다.

```jsx
const chemists = people.filter(person => person.profession === 'chemist')
const listItems = chemists.map(person =>
    <li>
        <img
        src={getImageUrl(person)}
        alt={person.name}
      />
    </li>
)
```

>화살표 함수는 암시적으로 `=>` 바로 뒤에 식을 반환하기 때문에 `return` 문이 필요하지 않다.

```jsx
const listItems = chemists.map(person =>
  <li>...</li> // 암시적 반환!
);
```

>하지만 **`=>` 뒤에 `{` 중괄호가 오는 경우 `return`을 명시적으로 작성해야 합니다!**

```jsx
const listItems = chemists.map(person => { // 중괄호
  return <li>...</li>;
});
```

`=> {` 를 표현하는 화살표 함수를 [“block body”](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions#function_body)를 가지고 있다고 말합니다. 이 함수를 사용하면 한 줄 이상의 코드를 작성할 수 있지만 `return` 문을 *반드시* 작성해야 합니다. 그렇지 않으면 아무것도 반환되지 않습니다!

## key로 목록의 항목 순서 유지하기

위 예제에서 (물론 이 때 목록의 각 자식에는 고유한 key prop이 필요하다는 에러가 뜬다.)라고 했다. **각 배열의 항목에는 고유하게 식별할 수 있는 문자열/숫자인 key가 필요하다.**

이 key는 각 컴포넌트가 어떤 배열 항목에 해당되는지 React에게 알려주는 역할을 한다. 만약 배열 항목이 수정될 경우(사이에 새로운 데이터가 들어오거나 삭제되는 경우 등) 이 배열 항목에 무슨 수정이 일어났는지 React가 정확히 인지해야 DOM 트리를 올바르게 업데이트 할 수 있다. 

근본적으로 React가 동작하는 원리는 가상 DOM 개념을 사용해 변화가 있는 부분만 브라우저에게 알려 DOM을 업데이트 하기 때문이다.

즉석에서 key를 생성하는 대신 데이터 안에 key를 포함해야 합니다.

```jsx
export const people = [{
  id: 0, // JSX에서 key로 사용됩니다.
  name: 'Creola Katherine Johnson',
  profession: 'mathematician',
  accomplishment: 'spaceflight calculations',
  imageId: 'MK3eW3A'
}, {
  id: 1, // JSX에서 key로 사용됩니다.
  name: 'Mario José Molina-Pasquel Henríquez',
  profession: 'chemist',
  accomplishment: 'discovery of Arctic ozone hole',
  imageId: 'mynHUSa'
}, {
  id: 4, // JSX에서 key로 사용됩니다.
  name: 'Subrahmanyan Chandrasekhar',
  profession: 'astrophysicist',
  accomplishment: 'white dwarf star mass calculations',
  imageId: 'lrWQx8l'
}];

```

따라서 객체 배열 안에 id 프로퍼티와 값을 넣어주고, 렌더링 되는 배열에 key를 넣어준다. 

## **각 항목이 여러 개의 DOM 노드를 렌더링 해야 하는 경우**

<></> 로는 key를 전달할 수 없으므로 단일

로 그룹화하거나, 구문을 사용한다. <></> 구문은 축약 형태로 별도의 속성을 지정할 수 없기 때문에 보다 명시적인 Fragment를 사용해야 한다.

```jsx
import { Fragment } from 'react';

// ...

const listItems = people.map(person =>
  <Fragment key={person.id}>
    <h1>{person.name}</h1>
    <p>{person.bio}</p>
  </Fragment>
);
```

## **key는 가져오는곳, 규칙**

데이터베이스에서 가져오거나, 클라이언트 개발 환경에서 생성해 부여할 수 있다. 

(1씩 up한다든지) 단, key={Math.random()}처럼 코드를 렌더링하는 부분에서 즉석에서 key를 생성하는 것은 지양해야 한다.

 key는 형제간에 고유해야 하며 변경되지 않아야 한다. (React가 변화를 인지하는 지표가 되므로)

또한 컴포넌트는 key를 prop로 받지 않는다. key로 사용하는 id나 식별자가 필요한 경우 별도로 전달해야 한다.

```
<Profile key={id} userId={id} />
```

**배열에서 항목의 인덱스를 key로 사용하는 경우가 있으나 이를 지양해야 한다.** 개발자가 key를 별도로 지정하지 않으면 React는 인덱스를 key로 사용한다. 하지만 인덱스를 key로 사용하면 다음과 같은 문제점이 생긴다.

1. 고유성 보장의 어려움: 각 항목에 대한 고유한 key를 보장할 수 없다. 배열에서 항목이 추가, 제거, 재정렬 되는 경우가 그렇다.
2. 성능 이슈: 인덱스를 Key로 사용하면 배열의 순서가 바귈 떄마다 모든 엘리멘트에 대해 불필요한 리렌더링이 발생할 수 있다.

따라서 각 항목에 고유한 id나 식별자를 사용하는 것을 권장한다.