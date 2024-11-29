# 객체 State 업데이트하기

state는 기본적으로 읽기전용이여서 불변성을 가집니다. .

지금까지 겉보기로는

```javascript
const [x, setX] = useState(0);
setX(5);
```

를 했을때, x의값이 0에서 5로 바뀌것처럼 보였으나. 실제로는 변하지 않고, 리액트가 5를 리렌더링을 통하여 화면에 그렇게 반영할 뿐이죠.

그래서 이런 원시값들(변경 불가능한 값)들을 다르게 사용하기 위해서, 교체하는 방식으로 새로운 값을 생성합니다.

```javascript
let x = 0;
x = 5; // 0이 5로 교체됨. (0은 변하지 않음)
```

위 예시에서 x = 5는 x 값을 새로 할당하는 방식이지, 기존의 값 0을 직접 수정한 것은 아닙니다.

## 객체 값은 어떠한가

객체 값도 마찬가지에요. 다시, state는 읽기전용이여서 값은 안바뀝니다. 그래서 우리는 교체를 해야해요.

```javascript
const [position, setPosition] = useState({
  x: 0,
  y: 0,
});
```

해당 state가 있다고 했을 때, 우리는 position값을 어떻게 조작 할 수 있을까요?

```javascript
onPointerMove={e => {
  position.x = e.clientX;
  position.y = e.clientY;
}}
```

이런식으로 쓴다면, 리액트는 객체가 변경되었는지 알 수 없습니다. 따라서 set함수를 사용해야 리액트가 state값이 바뀐걸 알고 state값이 바뀌면, 화면을 리렌더링해서 그 값으로 표현하죠.

```javascript
onPointerMove={e => {
  setPosition({
    x: e.clientX,
    y: e.clientY
  });
}}
```

정상적으로 set함수를 쓴 객체 교체입니다. 이 경우에 이 set함수는 React에게 다음과 같은 요청을 내립니다.

1. position을 새로운 객체로 교체해주세요
2. UI를 다시 렌더링해주세요
   => 그럼 이제 화면에서 무언가가 바뀐 것을 볼 수 있을겁니다.

## 전개 문법으로 객체 복사하기

우리는 어느 순간 폼에서 단 한 개의 필드만 수정하고, 나머지 모든 필드는 이전 값을 유지하고 싶을 수 있습니다.

```javascript
const [person, setPerson] = useState({
  firstName: "Barbara",
  lastName: "Hepworth",
  email: "bhepworth@sculpture.com",
});
```

여기서 어느 한 코드만 바꾸고 나머지 코드는 유지하고싶다?
=> 이 상황에서도 우리는 '교체'한다 라는 생각을 유지하고, 같은 객체에서 하나의 값만 바꿔서 객체를 교체해주면 됩니다.
그냥 단순하게 firstName만 바꾸고싶다고 setPerson(firstName:e.target.value) 이렇게 쓰면 안된다는거죠 나머지도 써주어야하는데, 이때 우린 전개연산자를 통하여 얕은복사를 이용할 수 있습니다.

```javascript
setPerson({
  ...person,
  firstName: e.target.value,
});
```

이렇게요.

```javascript
const [person, setPerson] = useState({
  name: "Niki de Saint Phalle",
  artwork: {
    title: "Blue Nana",
    city: "Hamburg",
    image: "https://i.imgur.com/Sd1AgUOm.jpg",
  },
});
```

이렇게 객체 내부에 객체가 있는 객체는?

```jsx
setPerson({
  ...person, // 다른 필드 복사
  artwork: {
    // artwork 교체
    ...person.artwork, // 동일한 값 사용
    city: "New Delhi", // 하지만 New Delhi!
  },
});
```

마찬가지로 이렇게 바꿔줄 수 있습니다.
