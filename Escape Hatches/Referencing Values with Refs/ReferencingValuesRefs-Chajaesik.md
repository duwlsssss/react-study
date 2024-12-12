### 이번시간에는 React에서 제공하는 hook중의 하나인 Reft에 대해 알아보고자한다.

 

#### 컴포넌트가 일부 정보를 "기억"하고 싶지만, 해당 정보가 렌더링을 유발하지 않도록 하려면 ref를 사용하자

#### 일반적으로 컴포넌트가 React를 "외부"와 외부 API--컴포넌트의 형태에 영향을 미치지 않는 브라우저 API와 통신해야할 때 ref를 사용한다.

#### 즉, 컴포넌트가 일부 값을 저장해야 하지만 렌더링 로직에 영향을 주지 않는 경우 refs를 선택한다.

 

### 학습내용
- 컴포넌트 ref를 어떻게 추가하는가
- ref의 값이 어떻게 업데이트 되는가
- ref가 state와 어떻게 다른가
- ref를 어떻게 안전하게 사용할까
 

 

### 기본 선언
```import { useRef } from 'react;```
 

#### 컴포넌트 내에서 useRef Hook을 호출하고 참조할 초깃값을 유일한 인자로 전달한다.

#### 아래는 값 0 에 대한 ref이다.

```const ref = useRef(0);```
 

#### 그러면 useRef는 다음과 같은 객체를 반환한다

```
{
	current: 0 // useRef에 전달한 값
}
```
 

#### 이렇게 선언된 ref는 ref.current 프로퍼티를 통해 해당 ref의 current 값에 접근할 수 있다.

#### 이 값은 의도적으로 변경할 수 있으므로 읽고 쓸 수 있다. 

#### 따라서 선언을 'let'으로 선언한다.

```
let ref = useRef(0);

ref.current = ref.current + 1;
```

### 현재는 0인 숫자를 가리키지만, state처럼 문자열, 객체, 함수 등 모든 것을 가리킬 수 있따.

#### state와 달리 ref는 읽고 수정할 수 있는 current 프로퍼티를 가진 일반 JS객체이다.


### 맨 처음 설명했듯, ref를 사용한 컴포넌트는 모든 증가에 대하여 다시 렌더링 되지 않는다.

 

### 사용 예시 ) 스톱워치 작성하기
#### ref와 state를 단일 컴포넌트로 결합할 수 있다.

#### 사용자가 "시작"을 누른 후 시간이 얼마나 지났는지 표시하려면 시작 버튼을 누른 시기와 현재 시각을 추적해야 한다.

#### 이 정보는 렌더링에 사용되므로 state를 유지해야하므로 다음과 같이 선언한다.
```
const [startTime, setStartTime] = useState(null); // 시작 시간
const [now, setNow] = useState(null); // 얼마나 지났지?! 계산을 위한
 ```

#### 시간을 추적하는 함수는 다음과 같이 작성한다.
```
    setInterval(() => {
      // 10ms 마다 현재 시간을 업데이트 합니다. 
      setNow(Date.now());
    }, 10);
 ```

#### 이때 "Stop" 버튼을 누르면 now state 변수의 업데이트를 중지하기 위해 기존의 interval를 취소해야한다.

#### 이를 위해서는 구별하기 위한 ID값이 필요한데, 

#### 따라서 setInterval 호출로 반환된 interval ID를 제공해야한다.

#### interval ID는 어딘가에 보관해야 하는데, 렌더링에 사용되지 않으므로 ref에 저장하는 것이 효율적일것이다.

#### (실제로 리렌더링 될 필요가 없기에)

```
function handleStart() {
	setStartTime(Date.now());
    setNow(Date.now());
    
    clear(intervalRef.current);
    intervalRef.current = setInterval(() => {
    	setNow(Date.now());
    },10);
 }
 ```
#### 우리는 렌더링에 정보를 사용할 때 해당 정보를 state로 유지하였다.

#### 이벤트 핸들러에게만 필요한 정보이고 변경이 일어날 때 리렌더링이 필요하지 않다면 ref를 사용하는 것이 효율적이라는 사실을 알게되었다.

 

### useRef는 내부적으로 어떻게 동작하는가?
```
function useRef(initialValue){
	const [ref, unused] = useState({ current: initialValue });
    return ref;
 }
 ```
### 첫번째 렌더 중에 useRef는 `{ current: initialValue }`을 반환한다.

#### 이 객체는 React에 의해 저장되므로 다음 렌더 중에 같은 객체가 반환된다.(항상 동일한 객체 반환)

### ref와 state의 차이

|      refs       |                state                |
|------------------|-------------------------------------|
| `useRef(initialValue)`는 `{ current: initialValue }`을 반환한다. | `useState(initialValue)`은 state의 현재 값과 setter 함수 `[value, setValue]`를 반환한다. |
| state를 바꿔도 리렌더 되지 않는다. | state를 바꾸면 리렌더 된다. |
| Mutable - 렌더링 프로세스 외부에서 current 값을 수정 및 업데이트할 수 있다. (따라서 let으로 선언) | "Immutable" -- state를 수정하기 위해서 state 설정 함수(setState)를 반드시 사용하여 리렌더 대기열에 넣어야 한다. |
| 렌더링 중에는 current 값을 읽거나 쓰면 안된다. (원본 값이 변경되므로 정보의 불일치를 야기) | 언제든지 state를 읽을 수 있다. 그러나 각 렌더마다 변경되지 않는 자체적인 state의 snapshot이 있다. |
