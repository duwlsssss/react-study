# Importing and Exporting Components
컴포넌트의 가장 큰 장점은 재사용성으로 컴포넌트를 조합해 또 다른 컴포넌트를 만들 수 있다는 것이다.

 컴포넌트를 여러 번 중첩하게 되면 다른 파일로 분리해야 하는 시점이 생긴다.... 

## 루트 컴포넌트

- CRA에서는 앱 전체가 src/App.js에서 실행된다.

- 리액트에서 root 컴포넌트는 애플리케이션 내의 최상위 엘리먼트를 의미한다.

## 컴포넌트를 import하거나 export하는 방법

- export, import로 모듈성을 강화하고 컴포넌트를 재사용할 수 있다.

- 컴포넌트를 다른 파일로 이동하려면 세 가지 단계

    1. 컴포넌트를 넣을 JS 파일 생성

    2. 새로 만든 파일에서 함수 컴포넌트를 export

    3. 컴포넌트를 사용할 파일에서 import

이 때 export, import 하는 방식은 default / named 2가지가 있다.

- default export는 파일 내 1개만 존재하고,
    - default import를 사용할 경우 원한다면 import 키워드 뒤에 다른 이름으로 값을 가져올 수 있다.
- named export는 여러 개가 존재할 수 있다.
    - import/export하는 쪽 모두 사용하는 이름이 같아야 한다.
![alt text](image.png)

> 보편적으로 한 파일에서 하나의 컴포넌트만 export 할 때 default export 방식을 사용하고, 여러 컴포넌트를 export 할 경우엔 named export 방식을 사용한다.

```jsx
function Profile() {
  return (
    <img
      src="https://i.imgur.com/QIrZWGIs.jpg"
      alt="Alan L. Hart"
    />
  );
}

export default function Gallery() {
  return (
    <section>
      <h1>Amazing scientists</h1>
      <Profile />
      <Profile />
      <Profile />
    </section>
  );
}

```

```jsx
import Gallery from './Gallery.js';

export default function App() {
  return (
    <Gallery />
  );
}

```

가끔 `.js`와 같은 파일 확장자가 없을 때도 있다

```jsx
import Gallery from './Gallery';
```

React에서는 `'./Gallery.js'` 또는 `'./Gallery'` 둘 다 사용할 수 있지만 

전자의 경우가native ES Modules 사용 방법에 더 가깝다.

ESM은 JS 모듈 시스템의 표준이며 파일 확장자를 명시적으로 쓰지 않고도 모듈을 가져오거나 내보낼 수 있다.

> 따라서 위와 같이 생략할 수 있으나, 파일 확장자를 명시하는 것을 권장한다.