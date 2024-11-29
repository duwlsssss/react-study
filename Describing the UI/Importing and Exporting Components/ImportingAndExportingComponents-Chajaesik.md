# 컴포넌트의 가장 큰 장점 - 재사용성
선언한 컴포넌트를 조합해 또 다른 컴포넌트를 만들 수 있다.

컴포넌트를 여러 번 중첩하게 되면 다른 파일로 분리하는 것을 권장하는데, 나중에 파일을 찾기 더 쉽고 재사용하기 편하기 때문이다.

React를 실행하면 기본적으로 `App.js`가 실행되는데, 이것은 root 컴포넌트 파일로 기본적으로 실행된다.

## 학습 내용
- Default와 Named Exports
- 한 파일에서 여러 컴포넌트를 import하거나 export 하는 방법
- 여러 컴포넌트를 여러 파일로 분리하는 방법

## Default와 Named Exports

JS에서는 default와 named export 두 가지 방법으로 값을 export한다. 한 파일에서 하나의 default export만 존재할 수 있고 named export는 여러 개가 존재할 수 있다.

| Syntax   | Export 구문                          | Import 구문                         |
|----------|--------------------------------------|-------------------------------------|
| Default  | `export default function Button() {}` | `import Button from './button.js';` |
| Named    | `export function Button(){}`         | `import { Button } from './button.js';` |

보통 한 파일에서 하나의 컴포넌트만 export한다면 default를, 여러 컴포넌트를 export 할 때는 named export 방식을 사용한다.

### 예시 코드

#### App.js
```javascript
import Gallery from './Gallery.js'; // default import
import { Profile } from './Gallery.js'; // named import

export default function App() {
  return (
    <Profile />
  );
}
```

#### Gallery.js
```javascript
export function Profile() { // Named export
  return (
    <img
      src="https://i.imgur.com/QIrZWGIs.jpg"
      alt="Alan L. Hart"
    />
  );
}

export default function Gallery() { // Default export
  return (
    <section>
      <h1>Amazing scientists</h1>
      <Profile />
      <Profile />
      <Profile />
    </section>
  );
}