# `useState`를 통해 `state` 변수를 설정하면 다음 렌더링이 큐에 들어간다.

 

#### 다음의 코드를 살펴보자 .

```javascript
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 1);
        setNumber(number + 1);
        setNumber(number + 1);
      }}>+3</button>
    </>
  )
}
 ```

#### 이전 포스트에서 다뤘듯, 버튼을 누르면 값은 1씩 증가한다.

##### 왜냐하면 각 렌더링의 state값은 고정되어있다고 배웠기 때문이다.

#### 하지만 !  여기에는 한 가지 요인이 더 있다.
##### 바로 React는 state 업데이트를 하기 전에 이벤트 핸들러의 모든 코드가 실행될 때까지 기다린다.

### 이 때문에 리렌더링은 모든 setNumber() 호출이 완료된 이후에만 일어난다.

#### 왜냐하면..너무 많은 리렌더링이 발생하지 않고 여러 컴포넌트의 나온 다수의 state 변수를 업데이트 할 수 있기 때문이다.

 

#### 하지만 !  이는 이벤트 핸들러와 그 안에 있는 코드가 완료될 때까지 UI가 업데이트되지 않는다는 의미이기도 하다.
 

##### "batching"(React는 이벤트 핸들러가 실행을 마친 후 state 업데이트를 처리하는 것)이라고 하는 이 위 동작은 React 앱을 훨씬 빠르게 실행할 수 있게 하고, <br>일부 변수만 업데이트된 "혼란"스러운 렌더링을 처리하지 않아도 된다.

### 다음 렌더링 전에 동일한 state 변수를 여러번 업데이트 하기

```javascript
export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(n => n + 1); //업데이터 함수 n => n + 1 함수를 큐에 추가합니다
        setNumber(n => n + 1); //업데이터 함수 n => n + 1 함수를 큐에 추가합니다
        setNumber(n => n + 1); //업데이터 함수 n => n + 1 함수를 큐에 추가합니다
      }}>+3</button>
    </>
  )
}
 ```

##### 이전의 코드(setNumber(number + 1)와 다르게, setNumber(n => n + 1) 와 같이 <br>이전 큐의 state를 기반으로 다음 state를 계한하는 함수를 전달함으로써 값을 "3"씩 증가 시킬 수 있다.

 

##### 절차적으로 살펴보자면, 다음과 같다.

 ![](image1.png)


 

 

#### 업데이트 후 state를 바꾸면 어떻게 될까 ?
 

```javascript
export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 5);
        setNumber(n => n + 1);
        setNumber(42);
      }}>Increase the number</button>
    </>
  )
}
 ```


#### 이벤트 핸들러가 실행하는 동안 React가 이 코드를 통해 작동하는 방식은 다음과 같다.

##### 1. `setNumber(number + 5) : number` 는 0이므로 `setNubmer(0 + 5)`이고 React는 "5로 바꾸기"를 큐에 추가한다.

##### 2. `setNumber(n => n + 1) : n => n + 1`는 업데이터 함수로, React는 이 함수를 큐에 추가한다.

##### 3. `setNumber(42)` : React는 "42로 바꾸기"를 큐에 추가한다.

 

![](image2.png)
 

### 따라서, React는 42를 최종 결과로 저장하고, useState에서 반환한다.

 

#### 이벤트 핸들러가 완료되면 React는 리렌더링을 실행한다.

#### 리렌더링하는 동안 React는 큐를 처리한다.

#### 업데이터 함수는 렌더링 중에 실행되므로, 업데이터 함수는 순수(값을 수정하지 않는)해야 하며 결과만 반환해야 한다.

 

### 명명 규칙
#### 업데이터 함수 인수의 이름은 해당 state 변수의 첫 글자로 지정하는 것이 일반적이다.

 
```javascript
setEnabled(e => !e);
setLastName(ln => ln.reverse());
setFriendCount(fc => fc * 2);

setEnabled(enabled => !enabled) //조금 더 자세한 코드
```