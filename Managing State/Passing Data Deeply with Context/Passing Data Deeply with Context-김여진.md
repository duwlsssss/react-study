### Context를 사용해 데이터를 깊게 전달하기

데이터를 필요한 컴포넌트까지 전달하기 위해 중간 단계의 컴포넌트들도 데이터를 받아야함. 불필요하게 props를 전달만 하는 컴포넌트가 발생 → 코드 가독성 저하, 유지보수 어려워짐 (props driiling)

React는 Context API를 제공해 Provider로 감싼 모든 자식 컴포넌트들을 리렌더링해 특정 데이터가 필요한 컴포넌트들에 데이터를 쉽게 전달 가능

1. Context 생성하고 내보내기

createContext는 초기값 매개변수 하나를 받음

```jsx
// LevelContext.js
import { createContext } from 'react';

export const LevelContext = createContext(1);
```

2. 데이터가 필요한 컴포넌트에 context 사용

`useContext` 훅으로 React에게 해당 컴포넌트가 읽으려하는 context 알림

```jsx
// Heading.js
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

export default function Heading({ children }) {
  const level = useContext(LevelContext);
  switch (level) {
    case 1:
      return <h1>{children}</h1>;
    case 2:
      return <h2>{children}</h2>;
    case 3:
    ...
  }
}
```
3. 데이터를 사용하는 컴포넌트에 context 제공하기

자식 컴포넌트들에게 context를 제공하기 위해 context provider로 감쌈
-> 어떤 컴포넌트가 특정 context를 요구하면 UI 트리에서 가장 가까운 Provider의 value를 넘김

```jsx
//Section.js
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

export default function Section({ children }) {
  const level = useContext(LevelContext);
  return (
    <section className="section">
      <LevelContext.Provider value={level+1}>
        {children}
      </LevelContext.Provider>
    </section>
  );
}

// 사용
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  // Section 내 Heading은 useContext(LevelContext)로 context를 요구하고, 가장 가까운 LevelContext.Provider의 value인 level+1을 받아 사용함
  return (
    <Section>
      <Heading>Title</Heading>
      <Section>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Section>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Section>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );
}
```

서로 다른 context는 분리돼 서로 영향을 주지 않음, 
상위에서 내려온 context를 재정의하는 유일한 방법은 하위 컴포넌트를 다른값을 가진 context provider로 감싸는 것 뿐임

- context 사용 사례
  - 테마 지정: 최상단에 context provider를 두고 시각적으로 조정돼야하는 컴포넌트에서 context 사용
  - 현재 계정 사용
  - 라우팅: 대부분의 라우팅 솔루션은 현재 경로를 유지하기 위해 내부적으로 context를 사용

- context 사용 전 고려할 것
  - props를 쓰는게 데이터 흐름을 분명하게 할 수도 있음 
  - jsx를 children으로 전달해서 props를 불필요하게 전달하기만 하는 컴포넌트 줄일 수 있음

  ```jsx
  function Layout({ posts }) {
    return (
      <div className="layout">
        <header>Header</header>
        <Posts posts={posts} />
        <footer>Footer</footer>
      </div>
    );
  }

  <Layout posts={posts} />

  //대신
  //Layout이 children prop을 받는 형식으로 수정

  function Layout({ children }) {
    return (
      <div className="layout">
        <header>Header</header>
        <main>{children}</main>
        <footer>Footer</footer>
      </div>
    );
  } 

  <Layout>
    <Posts posts={posts}>
  </Layout>
  ```

