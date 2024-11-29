# filter()와 map()을 사용하여 데이터 배열을 필터링하고 컴포넌트로 배열로 변환하기

## 학습 내용
- JS의 `map()`을 사용하여 배열을 컴포넌트로 렌더링하는 방법
- JS의 `filter()`를 사용하여 특정 컴포넌트만 렌더링하는 방법
- React에서 key가 필요한 경우

## 배열을 데이터로 렌더링하기

1. **리스트**

```javascript
<ul>
  <li>Creola Katherine Johnson: mathematician</li>
  <li>Mario José Molina-Pasquel Henríquez: chemist</li>
</ul>
```
2. **리스트 -> 배열**

```javascript
const people = [
  'Creola Katherine Johnson: mathematician',
  'Mario José Molina-Pasquel Henríquez: chemist',
];
```
3. **새로운 JSX 노드 배열인 listItems에 매핑**

```javascript
export default function List() {
  const listItems = people.map(person =>
    <li key={person}>{person}</li>
  );
  return <ul>{listItems}</ul>;
}
```

## 배열의 항목들을 필터링하기

#### 다음의 데이터 구조가 있다고 가정해보자

```javascript
const people = [{
  id: 0,
  name: 'Creola Katherine Johnson',
  profession: 'mathematician',
}, {
  id: 1,
  name: 'Mario José Molina-Pasquel Henríquez',
  profession: 'chemist',
}];
```

#### 위의 데이터 중에서 profession이 'chemist'인 사람들만 표시하고 싶을 때 filter() 메소드를 사용하여<br>해당하는 사람만 반환할 수 있다.이 메소드는 항목 배열을 받아 test(true 혹은 false)를 통과한 후 true가 반환된 새로운 배열을 반환한다.

```javascript
// 1. chemists 배열에 'chemist'인 조건이 true인 항목만 있는 새로운 배열 생성
const chemists = people.filter(person =>
  person.profession === 'chemist'
);

// 2. chemists를 매핑, 
const listItems = chemists.map(person => /* map 메소드를 이용하여 배열 요소에 접근
										 (이때 원본 배열은 수정되지 않음에 주의)
                                          유사한 컴포넌트 집합을 만들 때도 사용 */
     <img
       src={getImageUrl(person)}
       alt={person.name}
     />
);

// 3. 컴포넌트에서 listItems를 반환
return <ul>{listItems}</ul>;
```


Key를 사용하여 리스트 항목을 순서대로 유지

 

map() 호출 내부의 JSX 엘리먼트에는 항상 key가 필요하다

 

왜 ? -> key를 통해 각 컴포넌트가 어떤 배열 항목에 해당하는지 React에게 알려줘 올바르게 추론하고 DOM 트리에 업데이트하게 한다.(재정렬로 위치가 변경되더라도 key를 바탕으로 식별)

 

## Key 규칙

### 1. key는 형제 간에 고유해야 한다.

### 2. key는 변경되면 안된다.

 

## ! 주의사항 !

 

#### 1. 배열에서 항목의 인덱스를 key로 사용하면 안된다

| Index | Name |
|-------|------|
| 0     | A    |
| 1     | B    |
| 2     | C    |

만약 D를 A 다음에 추가한다면 다음과 같이 Index 1의 값이 B에서 D로 변경된다! 

| Index | Name |
|-------|------|
| 0     | A    |
| 1     | D    |
| 2     | B    |
| 3     | C    |

#### 2. key={Math.random}처럼 매번 키를 생성하면 렌더링 간에 Key가 일치하지 않아 모든 컴포넌트와 DOM이 매번 다시 생성될 수 있으며,<br> 이로인해 속도가 느려질뿐만 아니라 리스트 항목 내부의 모든 사용자 입력도 손실된다

따라서 데이터 기반의 안정적인 ID(uuid)를 권장
 