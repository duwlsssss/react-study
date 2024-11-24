### 1. Array State 업데이트 

state로 배열을 사용시 객체와 마찬가지로 업데이트 할때 복사본을 생성해 state setter에 넣어야 한다.

filter, map같은 배열 함수를 사용해 새 배열을 만들어 사용

| 작업 | 배열을 변경 | 새 배열을 반환 |
|------|-----------------|----------------|
| 추가 | push, unshift | concat, [...arr] |
| 제거 | pop, shift, splice | filter, slice |
| 교체 | splice, arr[i] = value | map |
| 정렬 | reverse, sort | 배열을 복사해 정렬 |

```jsx
const [artists, setArtists] = useState([]);

// ❌ 리렌더링 안 일어남, 다음 렌더링 state 스냅샷에 반영 안됨 
artists.push({
  id: nextId++,
  name: name,
});

// 🙆🏻‍♂️
setArtists(
  [
    ...artists, // 기존 배열의 모든 항목
    { id: nextId++, name: name } // 새 항목을 추가
  ]
);


// 항목 삭제
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


// 항목 반대로 정렬
function handleClick() {
  const nextList = [...artist];
  nextList.sort((a,b)=>a.id-b.id);
  setArtists(nextList);
}
<button onClick={handleClick}>오름차순</button>
```

immer 사용해 중첨된 Object value 수정 코드를 간결하게 작성 가능

```jsx
const initialList = [
  { id: 0, title: 'Big Bellies', seen: false },
  { id: 1, title: 'Lunar Landscape', seen: false },
  { id: 2, title: 'Terracotta Army', seen: true },
];

export default function BucketList() {
  const [myList, setMyList] = useState(initialList);

  function handleToggleMyList(artworkId) {
    setMyList(myList.map(artwork => {
      if (artwork.id === artworkId) {
        // 변경된 새 객체를 만들어 반환
        return { ...artwork, seen: !artwork.seen };
      } else {
        // 변경시키지 않고 반환
        return artwork;
      }
    }));
  }
  ...
}

export default function BucketList() {
  const [myList, updateMyList] = useState(initialList);

  function handleToggleMyList(artworkId) {
    updateMyList(draft => {
      const artwork = draft.find(a => a.id === artworkId);
      if(artwork) artwork.seen = !artwork.seen;
      //조건이 false라 변경사항 없으면 immer가 자동으로 원래 상태를 반환함
    });
  }
}
```
