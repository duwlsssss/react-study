# Import와 Export

# JSX (JavaScript XML)

JSX는 **JavaScript를 확장한 문법**으로, JS 파일에서 **HTML과 비슷한 마크업을 작성할 수 있게 해주는 문법**입니다.

## JSX의 사용 배경

웹이 인터랙티브해짐에 따라, 웹 페이지에서 **로직이 내용을 결정하는 경우**가 많아졌습니다. 이에 따라 **JavaScript가 HTML을 담당**하게 되었고, 리액트에서는 **마크업과 로직을 함께 위치시키기 위해 JSX**를 사용하게 되었습니다. 예를 들어, 버튼의 렌더링 로직과 마크업을 함께 두면, 이들 간의 **동기화 상태**를 쉽게 유지할 수 있습니다.

## JSX의 규칙

### 1. 하나의 루트 엘리먼트로 반환

- 한 컴포넌트에서 여러 요소를 반환하려면 반드시 하나의 부모 태그로 감싸야 합니다.
- 예시:

  ```jsx
  <div>
    <h1>Hedy Lamarr's Todos</h1>
    <img
      src="https://i.imgur.com/yXOvdOSs.jpg"
      alt="Hedy Lamarr"
      className="photo"
    />
    <ul>...</ul>
  </div>
  ```

  만약 최상위 부모 태그에 <div>를 추가하고 싶지 않다면, Fragment 문법을 사용하여 부모 요소 없이 여러 엘리먼트를 반환할 수 있습니다. Fragment는 <>와 </> 사이에 요소들을 넣는 방식으로 사용됩니다.

```jsx
<>
  <h1>Hedy Lamarr's Todos</h1>
  <img
    src="https://i.imgur.com/yXOvdOSs.jpg"
    alt="Hedy Lamarr"
    className="photo"
  />
  <ul>...</ul>
</>
```

\*왜 그래야하나?
-JSX는 내부적으로 JavaScript 객체로 변환되기 때문에, 하나의 함수에서 두 개 이상의 객체를 반환할 수 없습니다.

### 2. 모든 태그는 닫아주어야 한다

JSX에서는 태그를 명시적으로 닫아야 함

```jsx
<>
  <img
    src="https://i.imgur.com/yXOvdOSs.jpg"
    alt="Hedy Lamarr"
    className="photo"
  />
  <ul>
    <li>Invent new traffic lights</li>
    <li>Rehearse a movie scene</li>
    <li>Improve the spectrum technology</li>
  </ul>
</>
```

### 3. 대부분을 camelCase로 작성한다

\*JSX는 JavaScript로 변환되며, JSX에서 작성된 **속성(attribute)**은 JavaScript 객체의 키가 되기도 하고, 변수로 읽는 경우도 있습니다.

\*JavaScript에서는 변수명에 예약어를 사용할 수 없기 때문에, camelCase로 속성 이름을 작성하는 것이 권장됩니다. -예를 들어, HTML에서는 class와 for를 사용하지만, JSX에서는 **className**과 **htmlFor**를 사용해야 합니다.

```jsx
<img
  src="https://i.imgur.com/yXOvdOSs.jpg"
  alt="Hedy Lamarr"
  className="photo"
/>
<label htmlFor="trafficLight">Traffic light</label>
```
