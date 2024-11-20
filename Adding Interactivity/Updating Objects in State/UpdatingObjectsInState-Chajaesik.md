# State에는 모든 종류의 자바스크립트 값을 저장할 수 있다.

 
```javascript
const [x, setX] = useState(0);
```

#### 위 코드는 "읽기 전용"을 의미하는 "불변성"을 가진다.

##### 따라서 값을 교체하기 위해서는 리렌더링이 필요하다

```javascript
setX(5);
```
 

#### x state는 0에서 5로 바뀌었지만, 숫자 0 자체는 바뀌지 않았다.

#### 이렇듯 숫자, 문자열, 불리언같이 자바스크립트에 정의되어 있는 원시 값들은 변경할 숭 ㅓㅄ다.

 

### 다음 예로, state에 다음의 객체가 있다고 생각해보자

 
```javascript
const [position, setPosition] = useState({ x: 0, y: 0 });
```

#### 객체 자체의 내용을 다음과 같이 바꿀 수 있다. 이것을 변경(mutation)이라 한다.

 
```javascript
position.x = 5;
 ```

#### 하지만 React state의 객체들이 기술적으로 변경 가능할지라도, 숫자, 불리언, 문자열과 같이 불변성을 가진 것처럼 다루어야 한다.

#### 즉 객체를 변경하는 대신 교체 해야한다.

 
```javascript
      onPointerMove={e => {
        position.x = e.clientX;
        position.y = e.clientY;
      }}
```

#### 위의 코드는 x와 y의 위치를 기술적으로 객체 자체의 내용을 변경(mutation)하였다.

#### 이 코드를 사용하였더니 의도한대로 동작하지 않는다.

### 그 이유는..


### position에 할당된 객체를 이전 렌더링에서 수정한다.

#### 그러나 React는 state 설정 함수가 없으면 객체가 변경되었는지 알 수 없다.

#### 따라서, 리렌더링을 발생시키려면 아래와 같이 새 객체를 생성하여 state 설정 함수로 전달해야 한다.

 
```javascript
onPointerMove = {e => {
	setPosition({
     x: e.clientX,
     y: e.clientY
    });
 }}
 ```
#### 이 코드를 실행하여 setPosition은 React에게 다음과 같이 요청한다.

 
- position을 이 새로운 객체로 교체하라
- 그리고 이 컴포넌트를 다시 렌더링하라

### 지역 변경(local mutation) :
#### 변경(mutation)은 이미 state에 존재하는 객체를 변경할 때만 문제가 된다.

#### 방금 만든 객체를 수정하는 것은 아직 다른 코드가 해당 객체를 참고하지 않기 때문에 괜찮다.


### 전개 문법으로 객체 복사하기
#### 이전 예시에서 position 객체는 현재 커서 위치에서 항상 새롭게 생성된다.

#### 하지만 새로 생성하는 객체에 존재하는 데이터를 포함시킬 수 있는데,

### 예를 들어 form에서 단 한개의 필드만 수정하고 이전 값을 유지하고 싶을 수 있다.

 ```javascript
  function handleFirstNameChange(e) {
    person.firstName = e.target.value;
  }
  
    <input
        value={person.firstName}
        onChange={handleFirstNameChange}
    />
 
```
#### 위 input 필드는 onChange 핸들러가 state를 변경하기 때문에 동작하지 않는다.

 

#### 방금 배운것처럼 새로운 객체를 생성하여 setPerson으로 전달해야 한다.

### 하지만, 단 하나의 필드가 바뀌었기 때문에 기존에 존재하는 다른 데이터를 복사해야한다.

 
```javascript
setPerson({
  firstName: e.target.value, // input의 새로운 first name
  lastName: person.lastName,
  email: person.email
}); 

또는 ...객체 전개 구문 사용

setPerson({
  ...person, // 이전 필드를 복사
  firstName: e.target.value // 새로운 부분은 덮어쓰기
});
``` 

#### ...전개 문법은 얕은 복사임에 유의하자.

 

 

### 중첩된 객체 갱신하기
#### 아래와 같이 중첩된 객체 구조가 있다고 가정해보자

 
```javascript
const [person, setPerson] = useState({
  name: 'Niki de Saint Phalle',
  artwork: {
    title: 'Blue Nana',
    city: 'Hamburg',
    image: 'https://i.imgur.com/Sd1AgUOm.jpg',
  }
});
 ```

### 만약 person.artwork.city의 값을 변경하고 싶을 때, 여러분은 어떻게 할 것 인가?

 

#### 아마도...

```javascript
person.artwork.city = 'New Delhi';
 ```

#### 위와 같이 생각하지 않았는가?

 

#### 하지만 !!!! 위에서 배운거  같이, 직접 접근하지말고 새로운 객체를 생성해서 접근해야한다
#### 왜? -> React에서는 state를 변경할 수 없는 것(읽기 전용)으로 다루어야 하기 때문이다.

 

#### 따라서 city를 바꾸기 위해서  먼저(이전 객체로 생성된 ) 새로운 artwork 객체를 생성한 뒤;, 그것을 가리키는 새로운 person 객체를 만들어야한다.

 

### 다음과 같이..

 
```javascript
const nextArtwork = { ...person.artwork, city: 'New Delhi' };
const nextPerson = { ...person, artwork: nextArtwork };
setPerson(nextPerson);
 ```

 

### 객체는 사실 중첩되어 있지 않다
#### 아래의 코드를 살펴보자 

 
```javascript
let obj1 = {
  title: 'Blue Nana',
  city: 'Hamburg',
  image: 'https://i.imgur.com/Sd1AgUOm.jpg',
};

let obj2 = {
  name: 'Niki de Saint Phalle',
  artwork: obj1
};

let obj3 = {
  name: 'Copycat',
  artwork: obj1
};
 ```

#### 만약 당신이 obj3.artwork.city만을 변경하려 값을 수정했다면 obj2.artwork.city와 obj1.city 둘 다에 영향을 미칠 것이다

#### 이는 obj3.artwork, obj2.artwork, obj1이 같은 객체이기 때문이다(이것들은 프로퍼티를 통해 서로를 "가리키는" 각각의 객체이다"

 

### 따라서 각각의 state가 중첩되어있다면.. Immer을 사용해보자 

 

#### 이것은 변경 구문을 사용할 수 있게 해주며 복사본 생성을 도와주는 라이브러리이다

 

#### 이것은 일반적인 변경과 다르게 이전 state를 덮어쓰지 않는다 !

```javascript
updatePerson(draft => {
  draft.artwork.city = 'Lagos';
});
```
 

### Immer를 사용하는 방법
#### 1. package.json에 dependencies로 use-immer을 추가

#### 2. npm install 실행

#### 3. import { useState} from 'react를 import { useImmer } from 'use-immer"로 교체


### 왜 React에서 state 변경은 권장되지 않는가?
 

#### 1. 디버깅

#### 2. 최적화

#### 3. 새로운 기능

#### 4. 요구사항 변화

#### 5. 더 간단한 구현