# Importing and Exporting Components
컴포넌트의 가장 큰 장점은 재사용성으로 컴포넌트를 조합해 또 다른 컴포넌트를 만들 수 있다는 것입니다. 컴포넌트를 여러 번 중첩하게 되면 다른 파일로 분리해야 하는 시점이 생깁니다. 이때 import와 export를 사용하면 컴포넌트를 다른 파일로 분리할 수 있습니다.

## Root 컴포넌트란?
```js
// App.js
function Profile() {
  return (
    <img
      src="https://i.imgur.com/MK3eW3As.jpg"
      alt="Katherine Johnson"
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
이 예시의 컴포넌트들은 모두 `App.js`라는 **root 컴포넌트 파일**에 존재합니다. 설정에 따라 root 컴포넌트가 다른 파일에 위치할 수도 있습니다. Next.js처럼 파일 기반으로 라우팅하는 프레임워크일 경우 페이지별로 root컴포넌트가 다를 수 있습니다.

## 컴포넌트를 import 하거나 export하는 방법
랜딩 화면을 변경하게 되어 위 예시의 프로필 사진을 다른 곳에서 사용하게 된다면 `Gallery` 컴포넌트와 `Profile` 컴포넌트를 root 컴포넌트가 아닌 다른 파일로 옮기는 게 좋습니다. 변경하면 재사용성이 높아져 컴포넌트를 모듈로 사용할 수 있습니다. 컴포넌트를 다른 파일로 이동하려면 세 가지 단계가 있습니다.

1. 컴포넌트를 추가할 JS파일 생성
2. 새로 만든 파일에서 함수 컴포넌트 export(default 또는 named export방식을 사용)
3. 컴포넌트를 사용할 파일에서 import(export 방식에 따라 default 또는 named로 import)

아래와 같이 `App.js` 파일에서 `Profile`과 `Gallery` 컴포넌트를 빼서 새로운 `Gallery.js` 파일로 옮겼을 때, `App.js`파일에서 `Gallery`를 `Gallery.js`파일에서 import해서 사용할 수 있습니다.

```js
// App.js
import Gallery from './Gallery.js';

export default function App() {
  return (
    <Gallery />
  );
}

```
```js
// Gallery.js
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
위 예시에서 컴포넌트들이 두 파일로 나뉘게 되었습니다.
1. `Gallery.js`
    - `Profile` 컴포넌트를 정의하고 해당 파일에서만 사용되기 때문에 export하지 않습니다.
    - Default 방식으로 `Galley` 컴포넌트를 export합니다.
2. `App.js`
   - Default 방식으로 `Gallery`를 `Gallery.js`에서 import 합니다.
   - Root `App` 컴포넌트를 default 방식으로 export 합니다.

- 가끔 `.js`와 같은 파일 확장자가 없을 때도 있습니다.
- React에서는 `./Gallery.js`와 `./Gallery` 둘 다 사용 가능하지만 전자의 경우가 native ES Modules 사용 방법에 더 가깝습니다.

### Default와 Named Exports
Export 하는 방식에 따라 import하는 방식이 정해져 있습니다. Default export로 한 값을 named import로 가져올 때 에러가 발생합니다.

| Syntax | Export 구문 | Import 구문 |
|---|---|---|
| Default | `export default function Button() {}` | `import Button from './button.js';` |
| Named | `export function Button() {}` | `import { Button } from './button.js';` |

- Default import를 사용하는 경우 `import`구문에서 `import Banana from './button.js'`라는 식으로 다른 이름으로 값을 가져올 수 있습니다.
- named import의 경우 양쪽 파일에서 사용하고자 하는 값의 이름이 같아야 하기 때문에 named import라 불립니다.

보편적으로 한 파일에서 하나의 컴포넌트만 export할 경우 default export를 사용하고, 여러 컴포넌트를 export할 경우에는 named export를 사용합니다.

## 한 파일에서 여러 컴포넌트를 import하거나 export하는 방법

앞의 예시에서 전체 갤러리가 아닌 하나의 `Profile`만 사용하고 싶을 때, `Profile` 컴포넌트만 export하면 됩니다. 하지만 `Gallery.js` 파일엔 default export를 이미 사용하고 있기 때문에,  두 개의 default export를 정의 할 수 없습니다. 이 경우 새로운 파일로 `Profile` 컴포넌트를 생성하여 default export를 사용하거나, named export를 사용할 수 있습니다. **named export의 경우 한 파일에서도 여러번 사용 할 수 있습니다.**

```js
//Gallery.js named export방식
export function Profile() {
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
```js
//App.js named export방식 import
import { Profile } from './Gallery.js';

export default function App() {
  return (
    <Profile />
  );
}
```
위 예시의 경우 named export를 활용하여 App.js에서 `Profile` 컴포넌트를 한번만 사용하는 모습입니다. 이전 예시와 달리 `Gallery` 컴포넌트를 활용하지 않아 `Profile`컴포넌트 3개와 `h1`태그 또한 `App.js`에서 랜더링 되지 않습니다.