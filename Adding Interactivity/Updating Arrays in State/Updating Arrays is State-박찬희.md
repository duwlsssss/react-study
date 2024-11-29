# 배열 State 업데이트 하기
배열은 다른 종류의 객체입니다. 따라서 배열 State 또한 불변성을 지켜야합니다.

state는 불변하기 때문에 재할당되는 것이 아닌 새로운 state로 교체되어야 합니다. 따라서 새 배열을 setState 함수에 전달해야 하며 새로운 배열을 만들기 위해 `filter()`와 `map()` 메소드를 활용하는 것이 좋습니다.

|  | 배열 변경| 새 배열 반환|
|---|---|---|
|추가|`push`,`upshift`|`concat`,`[...arr]` 전개 연산자|
|제거|`pop`,`shift`,`splice`| `filter`,`slice`|
|교체|`splice`,`arr[i] = ...` 할당|`map`|
|정렬|`reverse`,`sort`|배열을 복사한 이후 처리|

> `slice`와 `splice` 함수는 이름이 비슷하지만 매우 다름!

## 배열에 항목 추가하기
```jsx
import { useState } from 'react';

let nextId = 0;

export default function List() {
  const [name, setName] = useState('');
  const [artists, setArtists] = useState([]);

  return (
    <>
      <h1>Inspiring sculptors:</h1>
      <input
        value={name}
        onChange={e => setName(e.target.value)}
      />
      <button onClick={() => {
        setName('');
        artists.push({
          id: nextId++,
          name: name,
        });
      }}>Add</button>
      <ul>
        {artists.map(artist => (
          <li key={artist.id}>{artist.name}</li>
        ))}
      </ul>
    </>
  );
}
```
위 예제는 `push()`메소드를 활용하여 배열을 변경합니다. 이는 `setName('')`을 통해 리랜더링 되지만 올바르지 않은 방법입니다.
아래와 같이 바꿔봅시다
```jsx
setArtists( // 아래의 새로운 배열로 state를 변경합니다.
  [
    ...artists, // 기존 배열의 모든 항목에,
    { id: nextId++, name: name } // 마지막에 새 항목을 추가합니다.
  ]
);
```
두 예제는 기능은 같지만 `setArtists`를 활용하여 새로운 배열을 만들어낼 수 있습니다. 또한 배열이기 때문에 객체와 달리 추가할 항목의 위치가 맨 앞이 될 수도 있고 맨 뒤가 될 수도 있습니다.

## 배열에서 항목 제거하기
다음 예제는 `filter()`메소드를 활용하여 배열의 항목을 제거했습니다. 버튼을 누르면 배열 항목들이 제거됩니다.
```jsx
  const [artists, setArtists] = useState(아티스트)

        {artists.map(artist => (
          <li key={artist.id}>
            {artist.name}{' '}
            <button onClick={() => {
              setArtists(
                artists.filter(a =>
                  a.id !== artist.id
                )
              );
            }}>
              Delete
            </button>
          </li>
        ))}
```

## 배열 변환하기
```jsx
  let initialShapes = [
    { id: 0, type: 'circle', x: 50, y: 100 },
    { id: 1, type: 'square', x: 150, y: 100 },
    { id: 2, type: 'circle', x: 250, y: 100 },
  ];
  const [shapes, setShapes] = useState(
    initialShapes
  );

  function handleClick() {
    const nextShapes = shapes.map(shape => {
      if (shape.type === 'square') {
        // 변경시키지 않고 반환합니다.
        return shape;
      } else {
        // 50px 아래로 이동한 새로운 원을 반환합니다.
        return {
          ...shape,
          y: shape.y + 50,
        };
      }
    });
    // 새로운 배열로 리렌더링합니다.
    setShapes(nextShapes);
  }
  ```
이번 예제는 `map()`메소드를 활용해 새로운 좌표를 가진 새로운 배열을 만들어냅니다.

## 배열에 항목 삽입하기
`slice()`메소드를 사용하면 배열의 일부분을 잘라내어 새로운 배열을 만들어냅니다. 이를 통해 원하는 인덱스를 제거할 수도 있지만 `...` 배열 전개 구문과 함께 사용하여 원하는 항목을 배열에 삽입할 수도 있습니다.
```jsx
let nextId = 3;
const initialArtists = [
  { id: 0, name: 'Marta Colvin Andrade' },
  { id: 1, name: 'Lamidi Olonade Fakeye'},
  { id: 2, name: 'Louise Nevelson'},
];

export default function List() {
  const [name, setName] = useState('');
  const [artists, setArtists] = useState(
    initialArtists
  );

  function handleClick() {
    const insertAt = 1; // 모든 인덱스가 될 수 있습니다.
    const nextArtists = [
      // 삽입 지점 이전 항목
      ...artists.slice(0, insertAt),
      // 새 항목
      { id: nextId++, name: name },
      // 삽입 지점 이후 항목
      ...artists.slice(insertAt)
    ];
    setArtists(nextArtists);
    setName('');
  }
}
```

## 배열 뒤집기
`reverse()`와 `sort()`메소드는 원본 배열을 변경시켜 직접 사용할 수는 없지만 배열을 복사하여 새로운 배열을 만들고 새로운 배열을 변경시킨다면 가능합니다!

이번 예제는 `...`배열 전개 구문으로 배열을 복사한 후 새로운 배열을 통해 변경하는 방법입니다.
```jsx
const initialList = [
  { id: 0, title: 'Big Bellies' },
  { id: 1, title: 'Lunar Landscape' },
  { id: 2, title: 'Terracotta Army' },
];

export default function List() {
  const [list, setList] = useState(initialList);

  function handleClick() {
    const nextList = [...list]; // 새로운 배열 복사
    nextList.reverse(); // 복사된 배열 변경
    setList(nextList); // 새로운 배열 리랜더링(변경 후)
  }
}
```

## 배열 내부의 객체 업데이트하기(배열 내부 중첩된 객체)
배열은 객체와 마찬가지로 참조 타입입니다. 따라서 값이 콜스택에 저장되지 않고 메모리 힙에 저장되며 콜스택에는 그 메모리 힙의 참조 값이 저장됩니다.
아래 예시를 보면 초기 state가 같습니다. 두 목록의 state가 공유되었고 한 목록의 체크박스를 선택하면 다른 목록에도 영향을 미칩니다.
```jsx
import { useState } from 'react';

let nextId = 3;
const initialList = [
  { id: 0, title: 'Big Bellies', seen: false },
  { id: 1, title: 'Lunar Landscape', seen: false },
  { id: 2, title: 'Terracotta Army', seen: true },
];

export default function BucketList() {
  const [myList, setMyList] = useState(initialList);
  const [yourList, setYourList] = useState(
    initialList
  );

  function handleToggleMyList(artworkId, nextSeen) {
    const myNextList = [...myList];
    const artwork = myNextList.find(
      a => a.id === artworkId
    );
    artwork.seen = nextSeen; // 기존 항목을 변경!!
    setMyList(myNextList);
  }

  function handleToggleYourList(artworkId, nextSeen) {
    const yourNextList = [...yourList];
    const artwork = yourNextList.find(
      a => a.id === artworkId
    );
    artwork.seen = nextSeen; // 기존 항목을 변경!!
    setYourList(yourNextList);
  }

  return (
    <>
      <h1>Art Bucket List</h1>
      <h2>My list of art to see:</h2>
      <ItemList
        artworks={myList}
        onToggle={handleToggleMyList} />
      <h2>Your list of art to see:</h2>
      <ItemList
        artworks={yourList}
        onToggle={handleToggleYourList} />
    </>
  );
}

function ItemList({ artworks, onToggle }) {
  return (
    <ul>
      {artworks.map(artwork => (
        <li key={artwork.id}>
          <label>
            <input
              type="checkbox"
              checked={artwork.seen}
              onChange={e => {
                onToggle(
                  artwork.id,
                  e.target.checked
                );
              }}
            />
            {artwork.title}
          </label>
        </li>
      ))}
    </ul>
  );
}
```
`myNextList` 배열 자체는 새로운 배열이지만, 항목 자체는 `myList`원본 배열과 동일한 객체를 참조하고 있습니다. 따라서 `artwork.seen = nextSeen;`을 통해 원본 배열의 artwork 항목 또한 변경됩니다. 이 원본 배열은 `yourArtworks`에도 존재하므로 버그가 발생합니다.

이는 `map()`메소드를 활용하여 해결할 수 있습니다.
```jsx
setMyList(myList.map(artwork => {
  if (artwork.id === artworkId) {
    // 변경된 *새* 객체를 만들어 반환합니다.
    return { ...artwork, seen: nextSeen };
  } else {
    // 변경시키지 않고 반환합니다.
    return artwork;
  }
}));
```
위 예제를 통해 `map()`메소드로 새로운 배열을 생성하며 초기 artwork의 id가 클릭한 객체의 `artwork.id`인 artworkID와 같다면 새로운 객체를 만들어 반환하여 같은 객체를 참조하는 것을 피할 수 있습니다.

## Immer로 간결한 업데이트 로직 작성하기
위 예제도 중첩된 객체처럼 Immer를 사용할 수 있습니다.
Immer를 사용함으로써 손쉽게 복사본을 생성하여 처리할 수 있습니다.
```jsx
  const [myList, updateMyList] = useImmer(
    initialList
  );

  function handleToggleMyList(artworkId, nextSeen) {
    updateMyList(draft => {
      const artwork = draft.find(a =>
        a.id === artworkId
      );
      artwork.seen = nextSeen;
    });
  }
```
`draft`는 항상 수행한 변경 사항에 따라 처음부터 다음 state를 구성합니다. 이 때문에 state를 변경하지 않고 변경 함수들도 적용 할 수 있습니다.