# 컴포넌트 간 state 공유하기

두 컴포넌트의 state가 항상 함께 변경되기를 원할 수 있습니다. 
그렇게 하려면 각 컴포넌트에서 state를 제거하고 가장 가까운 공통 부모 컴포넌트로 옮긴 후 props로 전달해야 합니다. 
이걸 state 끌어올리기라고 하며 React 코드 작성 시 가장 흔히 하는 일 중 하나입니다.

## State 끌어올리기
```jsx
import { useState } from 'react';

function Panel({ title, children }) {
  const [isActive, setIsActive] = useState(false);
  return (
    <section className="panel">
      <h3>{title}</h3>
      {isActive ? (
        <p>{children}</p>
      ) : (
        <button onClick={() => setIsActive(true)}>
          Show
        </button>
      )}
    </section>
  );
}

export default function Accordion() {
  return (
    <>
      <h2>Almaty, Kazakhstan</h2>
      <Panel title="About">
        With a population of about 2 million, Almaty is Kazakhstan's largest city. From 1929 to 1997, it was its capital city.
      </Panel>
      <Panel title="Etymology">
        The name comes from <span lang="kk-KZ">алма</span>, the Kazakh word for "apple" and is often translated as "full of apples". In fact, the region surrounding Almaty is thought to be the ancestral home of the apple, and the wild <i lang="la">Malus sieversii</i> is considered a likely candidate for the ancestor of the modern domestic apple.
      </Panel>
    </>
  );
}
```
예시에서는 부모 컴포넌트인 `Accordion`이 두 개의 `Panel`을 랜더링 합니다.
각 `Panel` 컴포넌트는 콘텐츠 표시 여부를 결정하는 `isActive`상태를 가집니다.
하지만 한 패널의 버튼을 눌러도 다른 패널에는 영향을 미치지 않고 독립적입니다.
이제 한 번에 하나의 패널만 열리도록 변경하려 합니다. 두 번째 패널을 열기 위해서 첫 번째 패널을 닫아야 합니다.

두 패널을 조정하려면 다음 세 단계를 통해 부모 컴포넌트로 패널의 "State 끌어올리기"가 필요합니다.
1. 자식 컴포넌트의 state를 제거합니다.
2. 하드 코딩된 값을 공통 부모로부터 전달합니다.
3. 공통 부모에 state를 추가하고 이벤트 핸들러와 함께 전달합니다.

## 1. 자식 컴포넌트에서 state 제거하기
`Panel`의 `isActive`에 대한 제어권을 부모 컴포넌트에 줄 수 있습니다. 즉 부모 컴포넌트는 `isActive`를 `Panel`에 ``prop으로 전달합니다.
```jsx
const [isActive, setIsActive] = useState(false);
// state 제거 및 아래 코드로 Panel 컴포넌트에 isActive를 prop으로 추가 
function Panel({ title, children, isActive }) {
```
이제 `Panel`의 부모 컴포넌트는 prop을 내려줌으로써 `isActive`를 제어할 수 있습니다. 반대로 `Panel`컴포넌트는 제어할 수 없습니다.

## 2. 하드 코딩 된 데이터를 부모 컴포넌트로 전달하기

state를 올리려면, 조정하려는 두 자식 컴포넌트의 가장 가까운 공통 부모 컴포넌트에 두어야합니다.
예시에서는 `Accordion` 컴포넌트입니다. 
이 컴포넌트는 두 패널의 상위에 있고 props를 제어 할 수 있기 때문에 어느 패널이 활성화되었는지 알 수 있습니다. 
`Accordion` 컴포넌트가 하드 코딩된 값을 가지는 `isActive`를 두 패널에 전달하도록 만듭니다.

```jsx
import { useState } from 'react';

export default function Accordion() {
  return (
    <>
      <h2>Almaty, Kazakhstan</h2>
      <Panel title="About" isActive={true}>
        ...
      </Panel>
      <Panel title="Etymology" isActive={true}>
        ...
      </Panel>
    </>
  );
}
```

## 3. 공통 부모에 state 추가하기
상태 끌어올리기는 종종 state로 저장하고 있는 것의 특성을 바꿉니다.
이 케이스에선 한 번에 하나의 패널만 활성화되어야 합니다. 
이를 위해 공통 부모 컴포넌트인 `Accordion`은 어떤 패널이 활성화 된 패널인지 추적하고 있어야합니다. 
state 변수에 `boolean`값을 사용하는 대신, 활성화되어있는 `Panel`의 인덱스 숫자르 사용할 수 있습니다.
```jsx
const [activeIndex, setActiveIndex] = useState(0);
```
`activeIndex`가 `0`이면 첫 번째 패널이 활성화되고, `1`이면 두 번째 패널이 활성화 된 것입니다.
이제 각 `Panel`에서 버튼을 클릭하면 `activeIndex`를 변경해야합니다.
이는 `Accordion` 컴포넌트 내에서 정의되었기 때문에 이벤트 핸들러를 prop으로 전달하며 명시적으로 허용해야합니다.

```jsx
<>
  <Panel
    isActive={activeIndex === 0}
    onShow={() => setActiveIndex(0)}
  >
    ...
  </Panel>
  <Panel
    isActive={activeIndex === 1}
    onShow={() => setActiveIndex(1)}
  >
    ...
  </Panel>
</>
```
이제 `Panel`의 `button`은 `onShow`prop을 클릭 이벤트 핸들러로 사용할 수 있습니다.
아래는 완성된 예시입니다.
```jsx
import { useState } from 'react';

export default function Accordion() {
  const [activeIndex, setActiveIndex] = useState(0);
  return (
    <>
      <h2>Almaty, Kazakhstan</h2>
      <Panel
        title="About"
        isActive={activeIndex === 0}
        onShow={() => setActiveIndex(0)}
      >
        내용
      </Panel>
      <Panel
        title="Etymology"
        isActive={activeIndex === 1}
        onShow={() => setActiveIndex(1)}
      >
        내용
      </Panel>
    </>
  );
}

function Panel({
  title,
  children,
  isActive,
  onShow
}) {
  return (
    <section className="panel">
      <h3>{title}</h3>
      {isActive ? (
        <p>{children}</p>
      ) : (
        <button onClick={onShow}>
          Show
        </button>
      )}
    </section>
  );
}
```

## 각 state의 단일 진리의 원천
React 앱에서 많은 컴포넌트는 자체의 state를 가집니다.
일부 상태는 입력처럼 리프 컴포넌트(트리 맨 아래에 있는 컴포넌트)와 가깝게 "생존"합니다.
예를 들어 클라이언트 측 라우팅 라이브러리도 현재 경로를 React state로 저장하고 props로 전달하도록 구현되어 있습니다.

**각각의 고유한 state에 대해 어떤 컴포넌트가 "소유"할지 고를 수 있습니다.**
이 원칙은 "단일 진리의 원천"을 갖는 것으로 알려져 있습니다. 
모든 state가 한 곳에 존재하는 것이 아닌 그 정보를 가지고 있는 특정 컴포넌트에 있다는 것을 말합니다.
컴포넌트 간의 공유된 state를 중복하는 대신 그들의 공통 부모로 끌어올리고 필요한 자식에게 전달해야합니다.

작업이 진행되며 앱은 계속 변합니다. 각 state가 어디에 "생존"해야 할지 고민하는 동안 state를 아래로 이동하거나 다시 올리는 것은 흔한 일이고 이건 모두 과정의 일부입니다!