### 배열은 JS에서 변경이 가능하지만,. state로 저장할 때는 변경할 수 없도록 해야한다.

#### 객체와 마찬가지로 state에 저장된 배열을 업데이트 하고싶을 때는 새로운 배열을 생성(혹은 기존 배열의 복사본을 생성)한 뒤, 이 새 배열을 state로 두어 업데이트 해야한다.

 

#### 따라서 `arr[0] = 'bird'`처럼 배열 내부의 항목을 재할당해서는 안 되며, push()나 pop 같은 함수로 배열을 변경해선 안된다.

 ![](./images-chajaesik/image1.png)


#### 오른쪽에 있는 filter나 map 같은 state의 원본 배열을 변경시키지 않는 함수를 사용하여 원본 배열로부터 
#### 새 배열을 만들수 있고, 이후 이 새 배열들을 state에 설정해야 한다.

 

### 우선, 헷갈리기 쉬운 splice와 slice의 차이에 대해 설명하고 그 다음 선호에 대한 예시를 작성하고자 한다.

 

#### slice를 사용하면 배열 또는 그 일부를 복사할 수 있다.
#### splice는 배열을 변경한다(항목을 추가하거나 제거)
 

### 1. ...전개 연산자
```javascript
setArtists( // 아래의 새로운 배열로 state를 변경합니다.
  [
    ...artists, // 기존 배열의 모든 항목에,
    { id: nextId++, name: name } // 마지막에 새 항목을 추가합니다.
  ]
);

 또는 .. 
 
 setArtists([
  { id: nextId++, name: name }, // 추가할 항목을 앞에 배치하고,
  ...artists // 기존 배열의 항목들을 뒤에 배치합니다.
]);
 ```

#### 위의 전개 구문은 배열의 가장 뒤에 추가하는 `push()`의 기능을,

#### 아래 전개 구문은 배열의 가장 앞에 추가하는 `unshift()`의 기능을 수행한다.

 

### 2. 배열에서 항목 제거하기
 
```javascript
let initialArtists = [
    { id: 0, name: 'Marta Colvin Andrade' },
    { id: 1, name: 'Lamidi Olonade Fakeye'},
    { id: 2, name: 'Louise Nevelson'},
  ];

  export default function List() {
    const [artists, setArtists] = useState(
      initialArtists
    );
  
    return (
      <>
        <h1>Inspiring sculptors:</h1>
        <ul>
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
        </ul>
      </>
    );
  }
```
#### 여기서 집중해야하는 코드는 아래와 같다.

 
```javascript
setArtists(
  artists.filter(a => a.id !== artist.id)
);
 ```

### 이것은, artist.id와 ID가 다른 artists로 구성된 배열을 생성한다는 의미이다.

#### 즉 각 artist의 "Delete" 버튼은 해당 artist를 배열에서 필터링한 다음, 반환된 배열로 리렌더링을 요청한다.

#### (filter가 원본 배열을 수정하지 않는 다는 점을 확인)

 

### 배열 변환하기
#### 배열의 일부 또는 전체 항목을 변경하고자 한다면, map()을 사용하여 새로운 배열을 만들 수 있다.

#### map에 전달할 함수는 데이터나 인덱스(또는 둘다) 각 항목을 처리할 수 있다.

 

```javascript
import { useState } from 'react';

let initialShapes = [
  { id: 0, type: 'circle', x: 50, y: 100 },
  { id: 1, type: 'square', x: 150, y: 100 },
  { id: 2, type: 'circle', x: 250, y: 100 },
];

export default function ShapeEditor() {
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

### 배열 내 항목 교체하기
#### 위에서 설명한 arr[0] = 'bird'와 같은 할당은 원본 배열을 변경시키므로, 이 경우에도 map을 사용해야 한다.

 
```javascript
let initialCounters = [
    0, 0, 0
]

export default function CounterList() {
    const [counters, setCounters] = useState(
        initialCounters
    );

    function handleIncrementClick(index){
        const nextCounters = counters.map((c, i) => {
            if( i === index){
                return c + 1
            }
            else {
                return c
            }
        })
        setCounters(nextCounters);
    }

    return (
        <ul>
          {counters.map((counter, i) => (
            <li key={i}>
              {counter}
              <button onClick={() => {
                handleIncrementClick(i);
              }}>+1</button>
            </li>
          ))}
        </ul>
      );
}
 ```

#### 위 코드에서 handleIncrementClick은 이해가 되었지만, return부분에서 counters.map를 쓰는 이유가 궁금할것 같아 

#### 추가 설명을 하고자 한다.

##### counters에는 현재 [0, 0, 0] 배열이 들어가있으므로 map을 통해 각 배열의 할당된 값을 순회하기 위하여 map을 사용하였고, 그 값을 counter로, 인덱스를 i로 설정한 것이다.

 

### 배열에 항목 삽입하기
#### 처음과 끝이 아닌 위치에 항목을 삽입하고 싶을 때,  ...배열 전개 구문과 slice() 함수(배열의 일부분 잘라내기)를 사용한다.

#### 항목을 삽입하려면 

##### 1. 삽입 지점 앞에 자른 배열을 전개

##### 2. 새 항목과 원본 배열의 나머지 부분을 전개하는 배열을 만든다

 

#### 구현한 아래 코드를 보자.
```javascript
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
  
<input
	value={name}
	onChange={e => setName(e.target.value)}
/>
<button onClick={handleClick}>
	Insert
</button>
<ul>
	{artists.map(artist => (
  	<li key={artist.id}>{artist.name}</li>
	))}
</ul>
 ```

#### nextId는 let으로 선언되어, 추가할 때마다 +1씩 증가하고, insertAt은 1로 고정되어, 매번 두번째 항목에 추가된다.

 

### 배열을 뒤집거나 정렬하거나
#### ...전개 구문과 map(), filter() 같은 비-변경 함수들로만은 모든 일을 할 수 없다

#### 예를 들어 배열을 뒤집거나 정렬하고 싶을 때 JS의 revers() 및 sort() 함수는 원본 배열을 변경시킨다.

 

#### 그렇다면 ? 먼저 배열을 복사하고 변경해버리자(아무튼 원본아님)
 
```javascript
export default function List() {
    const [list, setList] = useState(initialList);

    function handleClick(){
        const nextList = [...list]; // 원본 배열 복사
        nextList.reverse();
        setList(nextList);
    }
}
 ```

#### 위 코드를 여려번 실행하면 두 개의 배열을 번갈아 보여주는 것이아닌,

#### 계속 새로운 배열이 생성됨에 유의하라

 

### 그리고,

```javascript
const nextList = [...list];
nextList[0].seen = true; // 문제: list[0]을 변경시킨다.
setList(nextList);
 ```

### 위 코드에서 처럼 직접 배열의 값을 변경하면 

#### nextList와 list는 서로 다른 배열이지만, nextList[0]과 list[0]은 동일한 객체를 가리킨다.

#### 따라서 nextList[0].seen 을 변경하면 list[0].seen도 변경된다. 이것은 state 변경이므로 피해야 한다.

#### 이전에 설명한 중첩된 JS 객체 업데이트처럼 새로운 객체를 생성하여 변경해야 한다.

 

 

### 배열 내부의 객체 업데이트 하기
 

#### 중첩된 state를 업데이트할 때, 업데이트하려는 지점부터 최상위 레벨까지의 복사본을 만들어야 한다.

 
```javascript
  const [myList, setMyList] = useState(initialList);
  const [yourList, setYourList] = useState(
    initialList
  );
  function handleToggleMyList(artworkId, nextSeen) {
    const myNextList = [...myList];
    const artwork = myNextList.find(
      a => a.id === artworkId
    );
    artwork.seen = nextSeen;
    setMyList(myNextList);
  }

  function handleToggleYourList(artworkId, nextSeen) {
    const yourNextList = [...yourList];
    const artwork = yourNextList.find(
      a => a.id === artworkId
    );
    artwork.seen = nextSeen;
    setYourList(yourNextList);
  }
 ```

#### 위 예시에서 두 개의 개별 artwork 목록들은 초기 state가 initialList로 서로 같다.

#### 이로 인해, 한 목록의 변경이 다른 목록에도 영향을 끼친다.

 

### 마치 아래 코드를 실행 했을 때...

```javascript
const myNextList = [...myList];
const artwork = myNextList.find(a => a.id === artworkId);
artwork.seen = nextSeen; // 문제: 기존 항목을 변경시킵니다.
setMyList(myNextList);
 ```

#### 따라서 map을 사용하여 새로운 배열을 만들어 수정하면, 이전 항목의 변경 없이 업데이트된 버젼으로 대체할 수 있다.

 
```javascript
setMyList(myList.map(artwork => {
  if (artwork.id === artworkId) {
    // 변경된 *새* 객체를 만들어 반환합니다.
    return { ...artwork, seen: nextSeen }; // ...전개 구문 사용(전체를 의미)
  } else {
    // 변경시키지 않고 반환합니다.
    return artwork;
  }
}));
 ```

### 객체와 마찬가지로 배열도 Immer을 사용하여 복사본을 생성하여 처리할 수 있다.

 
```javascript
  function handleToggleMyList(id, nextSeen) {
    updateMyList(draft => {
      const artwork = draft.find(a =>
        a.id === id
      );
      artwork.seen = nextSeen;
    });
  }
 ```

### `Immer`을 사용하면 artwork.seen = nextSeen과 같이 변경해도 괜찮다.
#### 이는 원본 state를 변경하는 것이 아닌, Immer에서 제공하는 draft 객체를 변경하기 때문이다.