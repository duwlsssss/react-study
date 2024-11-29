# 렌더링 리스트

여러 개의 유사한 구성요소를 표시하고 싶을 때, map또는 filter 메소드를 통해서 데이터를 필터링 또는 구성요소배열로 변환 할 수 있습니다.

다음과 같은 코드로 배열 내부요소를 리스트렌더링 할 수 있습니다.

```jsx
const people = [
  "Creola Katherine Johnson: mathematician",
  "Mario José Molina-Pasquel Henríquez: chemist",
  "Mohammad Abdus Salam: physicist",
  "Percy Lavon Julian: chemist",
  "Subrahmanyan Chandrasekhar: astrophysicist",
];

export default function List() {
  const listItems = people.map((person) => <li>{person}</li>);
  return <ul>{listItems}</ul>;
}
```

다음과 같은 코드로 조건부로 필터하여 리스트렌더링 할 수 있습니다.

```jsx
const chemists = people.filter(person =>
  person.profession === 'chemist'
); //[{
  id: 1,
  name: 'Mario José Molina-Pasquel Henríquez',
  profession: 'chemist',
},{
  id: 3,
  name: 'Percy Lavon Julian',
  profession: 'chemist',
}]

const listItems = chemists.map(person =>
  <li>
     <img
       src={getImageUrl(person)}
       alt={person.name}
     />
     <p>
       <b>{person.name}:</b>
       {' ' + person.profession + ' '}
       known for {person.accomplishment}
     </p>
  </li>
);

return <ul>{listItems}</ul>;

```

## 매핑을 할 때의 주의사항

map으로 리스트 렌더링시 항상 key가 필요합니다. 그 key는 해당 배열의 다른 항목들 사이에서 고유하게 식별되는 문자 또는 숫자로 지정합니다 => 보통 문서에 나와있는 예제처럼 id값으로 많이 쓰는 것같네요.

### 만약 하나의 항목이 아닌, 여러개의 DOM노드를 렌더링 해야하는 경우엔?

=> 그땐 fragment구문을 이용합니다.

```jsx
import { Fragment } from "react";

// ...

const listItems = people.map((person) => (
  <Fragment key={person.id}>
    <h1>{person.name}</h1>
    <p>{person.bio}</p>
  </Fragment>
));
```

## key에 대하여

### key가 필요한 이유?

key는 리액트에서 무슨 일이 있는지 추론하고, 형제 간에 항목을 고유하게 식별하여, DOM트리에 올바른 업데이트를 위해 필요합니다.

### key 규칙 및 주의점

1. 키는 즉석으로 만드는게 아닌 데이터에 포함되어야하며, 형제간에 고유해야합니다
2. 키는 변경되어서는 안됩니다.
3. 키를 배열의 index로 사용할 수 있으나, 언젠가 아이템이 추가되거나 삭제될 경우, 오류가 발생할 가능성이 있기에 권장되지 않습니다.
