# state 구조 선택하기

## state 구조화 원칙
state를 최적화 하지 않더라도 정상적인 프로그램을 작성 할 수 있지만, 오류가 발생할 확률을 줄이기 위해 state를 구조화하는 것이 좋고 다음과 같은 원칙을 가집니다.

## 1. 연관된 state 그룹화
```jsx
const [x, setX] = useState(0);
const [y, setY] = useState(0);
// 두 변수가 항상 같이 변경된다면, 단일 state로 통합!
const [position, setPosition] = useState({ x: 0, y: 0 });
```
위 예제는 마우스 커서의 움직임에 따라 x,y 좌표를 가져왔던 state입니다. 마우스 커서는 항상 x,y좌표를 가지며 두 값은 동시에 업데이트 됩니다. 따라서 `position`이라는 state로 통합할 수 있습니다.

필요한 state의 수를 모를 때, 데이터를 객체나 배열로 그룹화 하는 것도 유용합니다.
```jsx
const [position, setPosition] = useState({
   x: 0,
   y: 0,
   z: 0, //3차원 공간으로 바뀐다는 가정에 추가할 수 있음!
  });
```
이전 예제를 예시로 z축을 추가했지만, 예를 들어 주소를 예로 들면 어느 나라부터 시작할 것인지 도, 시, 읍, 면, 동이라는 기준을 객체에 추가할 수 있습니다.


## 2.state의 모순 피하기

```jsx
  const [text, setText] = useState('');
  const [isSending, setIsSending] = useState(false);
  const [isSent, setIsSent] = useState(false);

  async function handleSubmit(e) {
  e.preventDefault();
  setIsSending(true);
  await sendMessage(text);
  setIsSending(false);
  setIsSent(true);
  }
  ```
이번 예제는 text를 전송하는 버튼에 두가지 `isSending`과 `isSent`가 변경되는 상황입니다.
`handleSubmit`이벤트가 실행될 때, `isSending`과 `isSent`는 동시에 `true`가 되어서는 안되기 때문에 `typing`(초기값), `sending`,`sent` 세 가지 유효한 상태 중 하나를 가질 수 있는 `status` state변수로 대체하는 것이 좋습니다!

```jsx
  const [text, setText] = useState('');
  const [status, setStatus] = useState('typing');

  async function handleSubmit(e) {
    e.preventDefault();
    setStatus('sending');
    await sendMessage(text);
    setStatus('sent');
  }
```
## 3. 불필요한 state 피하기

랜더링 중 컴포넌트의 props나 기존 state 변수에서 일부 정보를 계산할 수 있다면 컴포넌트 state에 해당 정보를 넣지 않아야 합니다.
예를 들어 `firstName`과 `lastName`을 합치면 `fullName`이 되지만 `fullName`또한 state 변수로 관리하지 않아도 된다는 것입니다.

## 4. state의 중복 피하기
여러 상태 변수 간 또는 중첩된 객체 내에서 동일한 데이터가 중복될 경우 동기화를 유지하기 어렵습니다. 가능하다면 중복을 줄여야합니다.
```jsx
  const [items, setItems] = useState(initialItems);
  const [selectedItem, setSelectedItem] = useState(
    items[0]
  );
  ```
  다음과 같은 두 state가 있습니다. `selectItem`은 `items`목록의 하나와 동일한 객체입니다. 이럴 경우 `items`의 객체에서 변화가 있더라도 `selectedItem`에서는 변화를 감지하지 못할 수 있다는 것입니다.
  이럴 경우 아래 예제와 같이 수정할 수 있습니다.

  ```jsx
  const [items, setItems] = useState(initialItems);
  const [selectedId, setSelectedId] = useState(0);
  ```
  이제는 `items`의 항목들을 수정하더라도 `selectedId`또한 업데이트 될것입니다.

  ## 4. 깊게 중첩된 state 피하기
  깊게 중첩된 state는 업데이트하기 쉽지 않습니다. 가능하면 state를 평탄한 방식으로 구성하는 것이 좋습니다.

  ```jsx
  export const initialTravelPlan = {
  id: 0,
  title: '(Root)',
  childPlaces: [{
    id: 1,
    title: 'Earth',
    childPlaces: [{
      id: 2,
      title: 'Africa',
      childPlaces: [{
        id: 3,
        title: 'Botswana',
        childPlaces: []
      },
      ...
```
다음은 여행 계획을 위해 방문할 곳들을 관리하기 위한 앱의 객체입니다. 이 앱에서 삭제하는 버튼을 추가하고 싶지만 객체는 끊임없이 깊어지고 있습니다. 중첩된 state를 업데이트하는 것은 변경된 부분부터 모든 객체의 복사본을 만드는 것을 의미합니다. 매우 비효율적입니다!!
따라서 쉽게 state를 업데이트하기 위해 평탄하게 만들어 보겠습니다. 지금 객체는 각 `place`가 자식 장소의 배열을 가지는 트리구조를 가지고 있습니다. 이것을 자식 장소 ID의 배열을 가지도록 변경하게된다면 각 장소 ID와 해당 장소에 대한 매핑을 할 수 있습니다.

```jsx
export const initialTravelPlan = {
  0: {
    id: 0,
    title: '(Root)',
    childIds: [1, 42, 46],
  },
  1: {
    id: 1,
    title: 'Earth',
    childIds: [2, 10, 19, 26, 34]
  },
  2: {
    id: 2,
    title: 'Africa',
    childIds: [3, 4, 5, 6 , 7, 8, 9]
  }, 
  3: {
    id: 3,
    title: 'Botswana',
    childIds: []
  },
  ...
  ```
  이제 장소를 제거하기 위해, state의 두 단계만 업데이트하면 됩니다.!
  - 업데이트 된 버전의 부모장소는 `childIds` 배열에서 제거된 ID를 제외해야 합니다.
  - 업데이트 된 버전의 루트 "테이블" 객체는 부모 장소의 업데이트 된 버전을 포함해야 합니다.

  ```jsx
  import { useState } from 'react';
import { initialTravelPlan } from './places.js';

export default function TravelPlan() {
  const [plan, setPlan] = useState(initialTravelPlan);

  function handleComplete(parentId, childId) {
    const parent = plan[parentId];
    // 부모 장소의 새 버전 만들기
    // 하위 id는 포함되지 않습니다.
    const nextParent = {
      ...parent,
      childIds: parent.childIds
        .filter(id => id !== childId)
    };
    // 루트 state 객체 업데이트
    setPlan({
      ...plan,
      // 업데이트 된 부모plan을 갖게 됩니다.
      [parentId]: nextParent
    });
  }

  const root = plan[0];
  const planetIds = root.childIds;
  return (
    <>
      <h2>Places to visit</h2>
      <ol>
        {planetIds.map(id => (
          <PlaceTree
            key={id}
            id={id}
            parentId={0}
            placesById={plan}
            onComplete={handleComplete}
          />
        ))}
      </ol>
    </>
  );
}

function PlaceTree({ id, parentId, placesById, onComplete }) {
  const place = placesById[id];
  const childIds = place.childIds;
  return (
    <li>
      {place.title}
      <button onClick={() => {
        onComplete(parentId, id);
      }}>
        Complete
      </button>
      {childIds.length > 0 &&
        <ol>
          {childIds.map(childId => (
            <PlaceTree
              key={childId}
              id={childId}
              parentId={id}
              placesById={placesById}
              onComplete={onComplete}
            />
          ))}
        </ol>
      }
    </li>
  );
}
```