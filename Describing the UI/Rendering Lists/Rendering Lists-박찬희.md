# Rendering Lists
데이터 모음에서 유사한 컴포넌트를 여러 개 표시하고 싶을 때, JavaScript 배열 메소드를 사용하여 데이터 배열을 조작할 수 있습니다. `filter()`와 `map()` 메소드를 활용하여 데이터 배열을 필터링하고 컴포넌트 배열로 변환하는 방법을 알아보겠습니다.

## 배열을 데이터로 렌더링하기
우리는 리스트를 랜더링할때 직접 데이터를 집어넣어 만들 수 있습니다. 하지만 동일한 데이터를 가지고 여러 인스턴스를 표시해야 하는 경우 JavaScript 객체와 배열에 저장하고 `map()`과 `filter()` 같은 메소드를 활용하여 컴포넌트 리스트를 랜더링 할 수 있습니다.


배열에서 항목 리스트를 생성하는 방법
1. 데이터를 베열로 이동
2. `people`의 요소를 새로운 JSX 노드 배열린 `listItem`에 매핑
3. `<ul>`로 매핑된 컴포넌트의 `listItems`를 반환
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
이렇게 하면 배열 데이터를 활용하여 리스트 랜더링을 할 수 있습니다.

## 배열의 항목들을 필터링하기

위 데이터를 더 구조화 해보겠습니다.
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
  id: 3,
  name: 'Percy Lavon Julian',
  profession: 'chemist',
}, {
  id: 4,
  name: 'Subrahmanyan Chandrasekhar',
  profession: 'astrophysicist',
}];
```
여기서 직업이 `chemist`인 사람들만 표시하고 싶다고 할때, JavaScript의 `filter()` 메소드를 사용하여 해당하는 사람만 반환 할 수 있습니다.

`filter()`메소드는 배열을 받아 `true` 혹은 `false`로 데이터를 분류해 `true`를 반환하는 항목만 있는 새로운 배열을 반환합니다.
이를 활용해 직업이 `chemist`인 사람만 필터링해보겠습니다.
1. `people`에서 `filter()`를 호출해 `person.profession === 'chemist'`로 필터링하여 `chemist`만 구성된 새로운 배열 `chemists`를 생성합니다.
2. 이후 `chemists`를 매핑합니다.
3. 마지막으로 컴포넌트에서 `listItems`를 반환합니다.
```jsx
import { getImageUrl } from './utils.js'; // 이해를 돕기 위해 이미지 url을 함수화 한 컴포넌트

export default function List() {
  //인물 데이터 배열
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
    id: 3,
    name: 'Percy Lavon Julian',
    profession: 'chemist',
  }, {
    id: 4,
    name: 'Subrahmanyan Chandrasekhar',
    profession: 'astrophysicist',
  }];

  //직업이 chemist인 사람들 필터링
  const chemists = people.filter(person =>
    person.profession === 'chemist'
  );
  //매핑
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
  //랜더링하기 위한 함수 반환
  return <ul>{listItems}</ul>;
}
```
> 화살표 함수는 암시적으로 화살표(`=>`) 바로 뒤에 식을 반환하기 때문에 `return`문을 생략할 수 있습니다. 하지만 `=>` 뒤에 `{` 중괄호가 오는 경우 `return`을 명시적으로 작성해야 합니다.

## key를 사용해서 리스트 항목을 순서대로 유지하기

위 예시들의 콘솔에 에러가 표시되는 것을 확인 할 수 있습니다.
```
Warning: Each child in a list should have a unique “key” prop.
```
각 배열 항목에 고유하게 식별할 수 있는 문자열 또는 숫자를 `key`로 지정해야 합니다.
```js
<li key={person.id}>...</li>
//map() 호출 내부의 JSX 엘리먼트에는 항상 key가 필요합니다!
```
Key는 각 컴포넌트가 어떤 배열 항목에 해당하는지 React에 알려주어 나중에 일치시킬 수 있도록 합니다. 이는 배열 항목이 정렬 등으로 인해 이동 및 삽입, 삭제 되는 경우 중요해집니다.

그렇기 때문에 데이터 안에 key를 포함해야 합니다. 위 예제를 데이터에 key 값을 포함하여 수정해보겠습니다.

```jsx
import { getImageUrl } from './utils.js'; // 이해를 돕기 위해 이미지 url을 함수화 한 컴포넌트

export default function List() {
  //인물 데이터 배열
  const people = [{
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
  id: 2, // JSX에서 key로 사용됩니다.
  name: 'Mohammad Abdus Salam',
  profession: 'physicist',
  accomplishment: 'electromagnetism theory',
  imageId: 'bE7W1ji'
}, {
  id: 3, // JSX에서 key로 사용됩니다.
  name: 'Percy Lavon Julian',
  profession: 'chemist',
  accomplishment: 'pioneering cortisone drugs, steroids and birth control pills',
  imageId: 'IOjWm71'
}, {
  id: 4, // JSX에서 key로 사용됩니다.
  name: 'Subrahmanyan Chandrasekhar',
  profession: 'astrophysicist',
  accomplishment: 'white dwarf star mass calculations',
  imageId: 'lrWQx8l'
}];

  //직업이 chemist인 사람들 필터링
  const chemists = people.filter(person =>
    person.profession === 'chemist'
  );
  //매핑
  const listItems = people.map(person =>
    <li key={person.id}>
      <img
        src={getImageUrl(person)}
        alt={person.name}
      />
      <p>
        <b>{person.name}</b>
          {' ' + person.profession + ' '}
          known for {person.accomplishment}
      </p>
    </li>
  );
  //랜더링하기 위한 함수 반환
  return <ul>{listItems}</ul>;
}
```

## `key`를 가져오는 곳
데이터 소스마다 다른 key 소스를 제공합니다.
- 데이터베이스의 데이터: 데이터베이스에서 데이터를 가져오는 경우 본질적으로 고유한 데이터베이스 key/ID를 사용할 수 있습니다.
- 로컬에서 생성된 데이터: 데이터가 로컬에서 생성되고 유지되는 경우(예: 메모 작성 앱의 노트), 항목을 만들 때 증분 일련번호나 `crypto.randomUUID()` 또는 `uuid` 같은 패키지를 사용하세요.

## key 규칙
- key는 형제 간에 고유해야합니다. 하지만 같은 key를 다른 배열의 JSX 노드에 동일한 key로 사용해도 괜찮습니다.
- key는 변경되어서는 안 되며 그렇게 되면 key는 목적에 어긋납니다! 렌더링 중에는 key를 생성하지 마세요.

## React에 key가 필요한 이유?

파일에 이름이 없다면 파일의 순서대로 참조할 것입니다. 큰 문제는 없어 보이지만 파일을 삭제한다면 두 번째 파일이 첫 번째 파일이 되고 세 번째 파일이 두 번째 파일이 되는 등 혼란스러워 질 수 있습니다.

이와 같은 혼란을 막기 위해 JSX key는 형제 항목 간 항목을 고유하게 식별 할 수 있습니다. 재졍렬로 인해 위치가 변경되더라도 `key`는 React가 생명주기 내내 해당 항목을 식별할 수 있게 해줍니다.

> 주의! <br> 
key를 전혀 지정하지 않으면 React는 인덱스를 사용합니다. 하지만 배열의 순서가 바뀌면 시간이 지남에 따라 랜더링 순서가 변경되고 이로 인해 버그가 발생할 수 있습니다. <br>
`key={Math.random()}`처럼 즉석해서 key를 상성하는 경우 랜더링 간에 key가 일치하지 않아 모든 컴포넌트와 DOM이 매번 다시 생성될 수 있습니다. 이는 속도 저하 및 리스트 항목 내부의 모든 사용자의 입력도 손실됩니다.<br>
`key`는 컴포넌트가 prop으로 받지 않습니다. 컴포넌트에 ID가 필요하다면 key 자체가 아닌 별도의 prop으로 전달해야합니다.