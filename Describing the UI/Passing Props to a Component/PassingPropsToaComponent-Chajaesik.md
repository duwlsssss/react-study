### React 모든 부모 컴포넌트는 props를 줌으로써 몇몇의 정보를 <br>자식 컴포넌트에게 객체, 배열, 함수를 포함한 모든 JavaScript 값을 전달할 수 있다.



# 학습 내용
- 컴포넌트에 props를 전달하고 읽는 방법
- 컴포넌트에 JSX를 전달하는 방법

### 컴포넌트에 props를 전달하고 읽는 방법

아래의 예시 코드에서 `Profile`(부모)은 `Avatar`(자식)에게 `person`과 `size`를 "내려"주고 있다.

```javascript
function Avatar({ person, size = 100 }) { // 자식 컴포넌트에서 props 읽기 (size 기본값 지정)
  return (
    <img
      className="avatar"
      src={getImageUrl(person)} 
      alt={person.name} // props 접근
    />
  );
}

export default function Profile() {
  return (
    <div>
      <Avatar // 자식 컴포넌트에 props 전달하기
        size={100}
        person={{ 
          name: 'Katsuko Saruhashi', 
          imageId: 'YfeOqp2'
        }}
      />
    </div>
  );
}
```

`Avatar`(자식)은 props를 다음과 같이 "내려" 받을 수 있다.
이 문법을 “구조 분해 할당”이라 한다.
```javascript
function Avatar({ person, size }) { // props를 선언할 때 ( 및 ) 안에 { 및 }에 주의
  // ...
}

function Avatar(props) { // 객체 참조 
  let person = props.person;
  let size = props.size;
  // ...
}
```
### JSX spread 문법으로 props 전달하기
```javascript
function Profile({ person, size, isSepia, thickBorder }) {
  return (
    <div className="card">
      <Avatar
        person={person}
        size={size}
        isSepia={isSepia}
        thickBorder={thickBorder}
      />
    </div>
  );
}
```

위 코드처럼 자식 컴포넌트에게 "모든" props를 전달할 때가 있다.

이럴 때 "spread"문법을 사용하면 간단히 다음과 같이 나타낼 수 있다.


```javascript
function Profile(props) {
  return (
    <div className="card">
      <Avatar {...props} /> // spread 문법, "모든 props를 전달할 때 사용하므로 주의"
    </div>
  );
}
```




### 자식을 JSX로 전달하기
JSX 내에 콘텐츠를 중첩하면, 부모 컴포넌트는 해당 콘텐츠를 children이라는 prop으로 받아 사용할 수 있다.



```javascript
function Avatar({ person, size }) {
  return (
    <img
      className="avatar"
      src={getImageUrl(person)}
      alt={person.name}
      width={size}
      height={size}
    />
  );
}

function Card({ children }) {  // children은 Avatar 컴포넌트를 의미
  return (
    <div className="card">
      {children}
    </div>
  );
}

export default function Profile() {
  return (
    <Card>
      <Avatar
        size={100}
        person={{ 
          name: 'Katsuko Saruhashi',
          imageId: 'YfeOqp2'
        }}
      />
    </Card>
  );
}
```



시간에 따라 props가 변하는 방식
아래의 Clock 컴포넌트는 color와 time 두 가지 props를 받는다.

#### 아래 코드는 컴포넌트가 시간에 따라 다른 props를 받을 수 있음을 보여준다.

```javascript
export default function Clock({ color, time }) {
  return (
    <h1 style={{ color: color }}> // 다른 color을 선택하면 변경
      {time} // 매초 변경
    </h1>
  );
}
```


### 이때 변경되는 시기는 주석과 같다.

#### Props는 컴포넌트의 데이터를 처음에만 반영하는 것이 아니라 모든 시점에 반영한다.



#### 그러나 props는 "변경할 수 없다"라는 의미의 불변성을 갖는데, 

#### 이는 컴포넌트가 props를 변경해야 하는 경우 기존의 props가 아닌 새로운 객체를 전달하도록 "요청"해야 한다.
#### 이것을 state를 설정한다고 한다.