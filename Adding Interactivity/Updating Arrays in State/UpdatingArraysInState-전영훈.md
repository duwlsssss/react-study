# 배열 State 업데이트하기

배열 state도 객체와 마찬가지로 업데이트하고싶을땐, 새 배열을 교체하는 식으로 작업을 해야합니다.

따라서 우리는 원본을 변경하지 않는, 새 배열을 반환하여 교체할 수 있도록, 전개연산자, filter, slice, map과같은 메소드를 주로 이용할 필요가 있습니다.

## 배열 추가하기

배열 추가는 보통 전개연산자를 많이씁니다. concat도 있지만, 더 편하니까요

```jsx
import { useState } from "react";

let nextId = 0;

export default function List() {
  const [name, setName] = useState("");
  const [artists, setArtists] = useState([]);

  return (
    <>
      <h1>Inspiring sculptors:</h1>
      <input value={name} onChange={(e) => setName(e.target.value)} />
      <button
        onClick={() => {
          setName("");
          setArtists([...artists, { id: nextId++, name: name }]);
        }}
      >
        Add
      </button>
      <ul>
        {artists.map((artist) => (
          <li key={artist.id}>{artist.name}</li>
        ))}
      </ul>
    </>
  );
}
```

## 배열 제거하기

배열제거는 filter를 주로 사용합니다. 보통 버튼을 눌렀을 때, 해당 데이터의 보통 id값으로 비교해서 없애는데요, 다음과 같은 코드입니다.

```jsx
function handleDeleteWatched(id) {
  setWatched((watched) => watched.filter((movie) => movie.imdbID !== id));
}

<button
  className="btn-delete"
  onClick={() => onDeleteWatched(movie.ImdbID)} //같은 key값을 인자로씀
>
  X
</button>;
```

watched라는 배열 데이터에서 만약 받아온 값의 id가 버튼이 렌더링될때의 데이터값의 id와 같으면 그걸 삭제하겠다는겁니다.

즉 배열내부에서 그 데이터만 없어진채로 새로운 배열이 생성되어 교체되는거죠.

## 배열 변환하기

보통 리스트를 돌릴때 많이 사용되는 map입니다. 리스트를 돌리면서 그 리스트 모두에 props를 붙이는 등으로 기존의 배열을 변환할 수 있습니다.

```jsx
{
  shapes.map((shape) => (
    <div
      style={{
        background: "purple",
        position: "absolute",
        left: shape.x,
        top: shape.y,
        borderRadius: shape.type === "circle" ? "50%" : "",
        width: 20,
        height: 20,
      }}
    />
  ));
}
```

## 항목교체하기

```jsx
import React, { useState } from "react";

export default function ChangeText() {
  const [items, setItems] = useState(["apple", "banana", "cherry"]);

  const handleChangeText = (indexToChange, newText) => {
    const updatedItems = items.map((item, index) => {
      if (index === indexToChange) {
        return newText; // 변경할 텍스트
      }
      return item; // 기존 항목은 그대로
    });
    setItems(updatedItems); // 상태 업데이트
  };

  return (
    <ul>
      {items.map((item, index) => (
        <li key={index}>
          {item}
          <button onClick={() => handleChangeText(index, "grape")}>
            Change
          </button>
        </li>
      ))}
    </ul>
  );
}
```

해당 코드에선 버튼을 클릭했을때, 그 클릭한 버튼이 'grape'로 교체되어 새로운 배열이 반환됩니다.
7
이외에도 배열 method들을 ( 삽입은 slice(n,n), 배열 인덱스 거꾸로 하는건 reverse( ) )를 활용하여 여러 배열 조작후 새롭게 반환이 가능합니다.
