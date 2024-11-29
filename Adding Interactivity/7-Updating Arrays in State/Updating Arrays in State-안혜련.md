# React에서 배열 State 업데이트하기
React에서 배열은 state로 사용할 수 있지만, 직접 수정하지 않고 항상 새로운 배열을 생성하여 업데이트해야 합니다. 
이를 통해 React는 변경 사항을 감지하고 컴포넌트를 올바르게 리렌더링할 수 있습니다.

## 배열 State 업데이트의 원칙
### 배열은 불변성을 유지해야 합니다.

- push()나 pop() 같은 배열 변경 메서드는 사용하지 않음.
- 대신 새로운 배열을 생성하여 state를 업데이트.
### 새 배열 생성 방법

- **배열 전개 구문 (...)**을 사용하여 기존 배열을 복사한 후 변경.
- filter(), map() 같은 함수로 새로운 배열 반환.


## 배열 업데이트 방법
### 1. 배열에 항목 추가

``` javascript
//- 잘못된 방법: push() 사용
artists.push({ id: nextId++, name: name }); // 원본 배열 수정

// - 올바른 방법: 배열 전개 구문 사용
setArtists([
  ...artists, // 기존 항목 유지
  { id: nextId++, name: name } // 새 항목 추가
]);
```

- 항목을 배열 앞에 추가

``` javascript
setArtists([
  { id: nextId++, name: name }, // 새 항목
  ...artists // 기존 항목
]);
```

### 2. 배열에서 항목 제거
- 방법: filter()를 사용하여 특정 조건을 만족하지 않는 항목만 포함한 새 배열 생성.

```javascript
setArtists(artists.filter(artist => artist.id !== id));
```

``` javascript
<button onClick={() => setArtists(artists.filter(a => a.id !== artist.id))}>
  Delete
</button>
```

### 3. 배열 항목 수정
- map()을 사용하여 조건에 맞는 항목만 업데이트한 새 배열 생성.

``` javascript

setArtists(
  artists.map(artist => {
    if (artist.id === id) {
      return { ...artist, name: 'Updated Name' }; // 업데이트된 새 객체
    }
    return artist; // 나머지는 변경하지 않음
  })
);
```

### 4. 배열 항목 삽입
- 특정 위치에 항목 삽입 시:

    - **slice()**와 배열 전개 구문 사용.

``` javascript
const insertAt = 1;
setArtists([
  ...artists.slice(0, insertAt), // 삽입 전까지의 배열
  { id: nextId++, name: 'New Artist' }, // 새 항목
  ...artists.slice(insertAt) // 삽입 이후의 배열
]);
```

### 5. 배열 변환
- map(): 배열의 항목을 변환하여 새 배열 생성.

- 예시: 모든 circle 타입 항목을 아래로 이동

``` javascript
setShapes(
  shapes.map(shape => {
    if (shape.type === 'circle') {
      return { ...shape, y: shape.y + 50 };
    }
    return shape; // 다른 항목은 그대로 유지
  })
);
``` 

- 배열 뒤집기 (reverse())

배열을 복사한 뒤 뒤집기.
``` javascript
setList([...list].reverse());
```

### 배열 내부 객체 업데이트
- 배열 안에 객체가 있는 경우, 객체는 배열 외부에 위치한 별도 참조이므로 직접 수정하면 안 됩니다.
- **map()**과 **객체 전개 구문 (...)**을 사용하여 안전하게 업데이트합니다.


``` javascript
//잘못된 방법

const updatedList = [...list];
updatedList[0].seen = true; // 원본 객체도 변경됨


// 올바른 방법
setList(
  list.map(item =>
    item.id === id
      ? { ...item, seen: true } // 새로운 객체로 교체
      : item // 나머지 항목은 그대로 반환
  )
);
```

- React 배열은 불변성을 유지해야 합니다.

    - push(), pop() 대신 전개 구문이나 함수(filter(), map()) 사용.
- 항목 추가, 삭제, 수정 방법

    - 추가: [...arr, newItem]
    - 삭제: filter()
    - 수정: map()과 객체 전개 구문.