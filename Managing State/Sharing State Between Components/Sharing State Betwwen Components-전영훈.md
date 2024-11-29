# 컴포넌트 간 State 공유하기

가끔 두 컴포넌트의 state가 함께 변경하길 원합니다. 근데 그게 만약 형제 요소 끼리라면, data flow상 같이 변경할 방법이 없겠죠. 그래서 그걸 부모 요소로 끌어 올려서 props로 받으면 됩니다. 이걸 lifting state up 라고 칭합니다

```jsx
import { useState } from "react";

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
        With a population of about 2 million, Almaty is Kazakhstan's largest
        city. From 1929 to 1997, it was its capital city.
      </Panel>
      <Panel
        title="Etymology"
        isActive={activeIndex === 1}
        onShow={() => setActiveIndex(1)}
      >
        The name comes from <span lang="kk-KZ">алма</span>, the Kazakh word for
        "apple" and is often translated as "full of apples". In fact, the region
        surrounding Almaty is thought to be the ancestral home of the apple, and
        the wild <i lang="la">Malus sieversii</i> is considered a likely
        candidate for the ancestor of the modern domestic apple.
      </Panel>
    </>
  );
}

function Panel({ title, children, isActive, onShow }) {
  return (
    <section className="panel">
      <h3>{title}</h3>
      {isActive ? <p>{children}</p> : <button onClick={onShow}>Show</button>}
    </section>
  );
}
```

해당 코드는 onClick을 통해 set함수를 지정하고, 그로인해 state값이 업데이트되어 children 이 보여지는 함수입니다. 다만, state값이 두 Panel 컴포넌트의 부모인 Accordion 컴포넌트에 존재하여, accordion 내부의 state의 영향을 공유받게 됩니다.
