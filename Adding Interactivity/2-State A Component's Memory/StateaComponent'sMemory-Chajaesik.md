# 컴포넌트는 상호 작용의 결과로 화면의 내용을 변경해야 하는 경우가 많다

#### 예를 들어 폼에 입력하면 입력 필드가 업데이트 되어야하고, 이미지 리스트에서 "다음"을 누르면 이미지가 변경되는 등..

#### 컴포넌트는 현재 입력값, 현재 이미지 같은 것을 "기억"해야하고, `React`는 이런 종류의 컴포넌트별 메모리를 `state`라 한다.

 
```javascript
export default function Gallery() {
  let index = 0;

  function handleClick() {
    index = index + 1;
  }

  let sculpture = sculptureList[index];
  return (
    <>
      <button onClick={handleClick}>
        Next
      </button>
      <h2>
        <i>{sculpture.name} </i> 
        by {sculpture.artist}
      </h2>
      <h3>  
        ({index + 1} of {sculptureList.length})
 ```

#### 위 코드를 실행하여, Next버튼을 눌러도 화면에 변화가 발생하지 않는다.  그 이유는 다음과 같다.

#### 현재 `let`이 지역변수로 선언되어 있기 떄문이다.

### 일반 변수로 충분하지 않은 경우
- 지역 변수는 랜더링 간에 유지되지 않는다
- 지역 변수를 변경해도 렌더링을 일으키지 않는다.

 

### 따라서 컴포넌트를 새로운 데이터로 업데이트하기 위해선 아래의 두 가지가 필요하다

- 렌더링 사이에 데이터를 유지
- `React`가 새로운 데이터로 컴포넌트를 렌더링하도록 유발

 

### 위 두 가지 조건을 충족하는 것이 `useState`이다. 

- 렌더링 간에 데이터를 유지하기 위한 `state` 변수

- 변수를 업데이트하고, React가 컴포넌트를 다시 렌더링하도록 유발하는 state setter함수

#### `useState`는 `"Hook"`이라 불리우는데, React에서 "use"로 시작하는 다른 모든 함수를 "훅"이라 한다.<br>(그렇기 때문에 커스텀 훅을 만들 때에도 use로 시작해야한다)

#### 훅(use로 시작하는 함수들)은 컴포넌트의 최상위 수준 또는 커스텀 훅에서만 호출할 수 있음에 주의하자

 

### 위에서 선언한 지역 변수를 `useState`로 선언해보면.. 

```javascript
let index = 0  // 일반 지역 변수

const [index, setIndex] = useState(0); // useState로 선언

```
`index`는 `state` 변수이고, `setIndex`는 `setter`함수이다.

여기서 **[ 와 ] 문법**을 **"구조 분해 할당"**이라 하고, `useState`가 반환하는 배열에는 항상 두개의 항목이 있다.

두번째 인자값에 set을 붙이는 **"규칙"**이 있다.

 

### 실제 작동 방식은 다음과 같다..
#### 컴포넌트가 처음 렌더링 된다 : (index의 초기값으로 useState를 사용해 0을 전달했으므로 [0, setIndex]를 반환한다 
#### React는 0을 최신 state 값으로 기억한다.
- state를 업데이트한다
  - 사용자가 버튼을 클릭하여 setIndex(index + 1)을 호출한다. 현재 index는 0이므로 setIndex(1)이며, 이는 React에 index가 1임을 기억하게 하고 이를 화면에 렌더링한다.
- 컴포넌트가 두번쨰로 렌더링 된다
  - React는 여전히 useState(0)을 보지만, index를 1로 설정한 것을 기억하고 있기 때문에 [1, setIndex]를 반환한다.
- 위의 과정을 반복한다.

### 컴포넌트에 여러 state 변수 지정하기
 
```javascript
export default function Gallery() {
    const [index, setIndex] = useState(0);
    const [showMore, setShowMore] = useState(false);

    function handleClick(){
        setIndex(index + 1);
    }

    function handleMoreClick(){
        setShowMore(!showMore);
    }

    <button onClick={handleClick}>
        Next
    </button>

    <button onClick={handleMoreClick}>
        {showMore ? 'Hide' : 'Show'} details
    </button>
}
```

#### 위 예시에서와 같이, `index`와 `showMore`처럼 서로 연관이 없는 경우 여러 개의 `state` 변수를 가지는 것이 좋다.

#### 하지만 두 `state` 변수를 자주 함께 사용하는 경우 두 변수를 하나로 합치는게 좋을 수도 있다.

 

 

##### 그렇다면 React는 어떤 state를 반환할지 어떻게 알 수 있을까?
##### useState 호출이 어떤 state 변수를 참조하는지에 대한 정보를 받지 못하는데, 
### useState에 전달되는 "식별자" 없이 어떤 변수를 반환할지 어떻게 알 수 있을까? 

##### 간결한 구문을 구현하기 위해 훅은 동일한 컴포넌트의 모든 렌더링에서 안정적인 호출 순서(항상 같은 순서)에 의존한다.

##### 내부적으로 React는 모든 컴포넌트에 대해 한 쌍의 state 배열을 가집니다. 또한 렌더링 전에 0으로 설정된 현재 인덱스 쌍을 유지한다. 
##### useState를 호출할 때마다, React는 다음 state 쌍을 제공하고 인덱스를 증가시킨다.

State는 격리되고 비공개로 유지된다.
 
```javascript
export default function Page() {
  return (
    <div className="Page">
      <Gallery />
      <Gallery />
    </div>
  );
}
```

##### 이전에 위 코드를 useState가 아닌 방법으로 실행했을 때에는 컴포넌트가 동일하게 변경되었지만,

##### `useState`는 화면에서 컴폰넌트에 지역적이므로 동일한 컴포넌트를 두번 렌더링하면 각 복사본은 "독립"된 state를 가진다.
##### 따라서 하나를 변경해도 다른 하나에는 영향을 미치지 않는다.