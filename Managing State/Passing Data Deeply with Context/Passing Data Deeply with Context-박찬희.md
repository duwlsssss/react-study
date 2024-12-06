# Context를 사용해 데이터를 깊게 전달하기

보통의 경우 부모 컴포넌트에서 자식 컴포넌트로 props를 통해 정보를 전달합니다. 그러나 중간에 많은 컴포넌트를 거쳐야 하거나, 애플리케이션의 많은 컴포넌트에서 동일한 정보가 필요한 경우에는 props를 전달하는 것이 번거롭고 불편할 수 있습니다.

context는 부모 컴포넌트가 트리 아래에 있는 모든 컴포넌트에 깊이에 상관없이 정보를 명시적으로 props를 통해 전달하지 않고도 사용할 수 있게 해줍니다.

## Prop drilling
- Prop Drilling: props를 하위 컴포넌트로 전달하는 용도로만 쓰이는, 즉 컴포넌트를 거치지만 특정 하위 컴포넌트로 전달하기 위해 props를 전달하는 과정을 말합니다.

이는 불필요한 코드를 증가시키고 단순 전달만 하는 것은 비효율적입니다.
따라서 이러한 비효율을 Context가 해결할 수 있습니다.

### Props 전달의 대안 Context
Context는 부모 컴포넌트가 그 아래의 트리 전체에 데이터를 전달할 수 있도록 해줍니다.
Context는 부모가 트리 내부 전체에, 심지어 멀리 떨어진 컴포넌트에조차 어떤 데이터를 제공할 수 있도록 합니다.

Context를 사용하기에는 크게 3가지 방법이 있습니다.
1. Context 생성하기
2. 데이터가 필요한 컴포넌트에서 Context 사용하기
3. 데이터를 지정하는 컴포넌트에서 Context 제공하기

## Context 생성하기

```jsx
//LevelContext.js
import { createContext } from 'react';

export const LevelContext = createContext(1);
```
우선 Context를 만들고 컴포넌트에서 사용할 수 있도록 파일을 내보냅니다.
`createContext`의 유일한 인자는 기본값입니다. 이 예시에서 `1`은 가장 큰 제목 레벨을 나타내지만 모든 종류의 값을 전달할 수 있습니다.

## Context 사용하기

React에서 `useContext` Hook과 생성한 Context를 가져옵니다.
```jsx
// App.js
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  //Context를 사용하므로써 Heading 컴포넌트에 level props를 내리지 않아도 됩니다
  return (
    //기존 <Heading level={n} ==> "level={n}" 제거! > 컴포넌트 별로 수정 된 상황
    <Section level={1}>
      <Heading>Title</Heading>
      <Section level={2}>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Section level={3}>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Section level={4}>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );
}

// Heading.js
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

export default function Heading({ children }) {
  // React에 Heading 컴포넌트가 LevelContext 읽으려한다고 알려주기
  const level = useContext(LevelContext); // 이전에 받아온 level prop을 useContext로 변경

  switch (level) {
    case 1:
      return <h1>{children}</h1>;
    case 2:
      return <h2>{children}</h2>;
    case 3:
      return <h3>{children}</h3>;
    case 4:
      return <h4>{children}</h4>;
    case 5:
      return <h5>{children}</h5>;
    case 6:
      return <h6>{children}</h6>;
    default:
      throw Error('Unknown level: ' + level);
  }
}

//Section.js
export default function Section({ children }) {
  return (
    <section className="section">
      {children}
    </section>
  );
}
```
하지만 이 예시는 아직 부족합니다. Context는 사용하고 있지만, 아직 제공하지 않았기 때문에 모든 제목의 크기는 동일합니다.

Context를 제공하지 않으면 React는 이전 단계에서 지정한 기본값을 사용합니다. 이 예시에서는 `1`을 `createContext`의 인수로 지정했으므로 `useContext(LevelContext)`가 `1`을 반환하고 모든 제목을 `<h1>`로 설정합니다.

이 문제를 각각의 `Section`이 고유한 context를 제공하여 해결합시다

## Context 제공하기

`Section`컴포넌트는 자식들을 랜더링하고 있습니다.<br/>
여기에 `LevelContext`를 자식들에게 제공하기 위해 context provider로 감싸줍니다.
```jsx
import { LevelContext } from './LevelContext.js'; // LevelContext 불러오기

export default function Section({ level, children }) {
  return (
    //context provider로 감싸기
    <section className="section">
      <LevelContext.Provider value={level}>
        {children}
      </LevelContext.Provider>
    </section>
  );
}
```

context provider는 React에게 `Section`내의 어떤 컴포넌트가 `LevelContext`를 요구하면 `level`을 주라고 알려줍니다. 컴포넌트는 그 위에 있는 UI트리에서 가장 가까운 `<LevelContext.Provider>`의 값을 사용합니다.

Context를 사용하기 전과 동일한 결과를 만들어 냈습니다. 하지만 `level` prop을 각 `Heading` 컴포넌트에 전달할 필요는 없습니다. 대신 위의 가장 가까운 `Section`에게 제목 레벨을 확인합니다. 이는 다음과 같이 진행됩니다.

1. `level` prop을 `<Section>`에 전달합니다.
2. `Section`은 자식을 `<LevelContext.Provider value={level}>`로 감싸줍니다.
3. `Heading`은 `useContext(LevelContext)`를 사용해 가장 근처의 `LevelContext`의 값을 요청합니다.


## 같은 컴포넌트에서 context를 사용하며 제공하기
위 예제들은 각각 Section에 `level`을 수동으로 지정해야합니다. <br>
Context를 통해 위의 컴포넌트에서 정보를 읽을 수 있으므로 각 `Section`은 위의 `Section`에서 `level`을 읽고 자동으로 `level + 1`을 아래로 전달할 수 있습니다.

```jsx
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

export default function Section({ children }) {
  const level = useContext(LevelContext); // Section에서도 LevelContext 읽으려한다 알려주기!
  return (
    <section className="section">
      <LevelContext.Provider value={level + 1}> 
        {children}
      </LevelContext.Provider>
    </section>
  );
}

// App.js
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  // 이젠 Section에도 prop을 전달할 필요가 사라졌습니다!
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
이제 `Heading`과 `Section` 모두 자신들이 얼마나 깊이 있는지 확인하기 위해 `LevelContext`를 읽습니다. 그리고 `Section`은 그 안에 있는 어떤 것이든 더 깊은 레벨이라는 것을 명시하기 위해 자식들을 `LevelContext`로 감싸고 있습니다.

## Context 사용하기 전에 고려할 것

1. Props 전달하기로 시작하기 - 여러 개의 props가 여러 컴포넌트를 거쳐 가는 것은 그리 이상한 일이 아닙니다. 이는 어떤 컴포넌트가 어떤 데이터를 사용하지는지 명확하게 해줍니다. 데이터의 흐름이 props를 통해 분명해져 코드를 유지보수 하기에도 좋습니다.

2. 컴포넌트를 추출하고 JSX를 children으로 전달하기 - 데이터를 사용하지 않고 많은 중간 컴포넌트를 지나 어떤 데이터를 전달하는 경우에는 컴포넌트 추출하는 것을 잊는 경우가 많습니다. `posts`처럼 직접 사용하지 않는 props를 `<Layout posts={posts} />`와 같이 전달할 수 있습니다. 대신 `Layout`은 `children`을 prop으로 받고 `<Layout><Posts posts={posts} /><Layout>`을 렌더링하세요. 이렇게 하면 데이터를 지정하는 컴포넌트와 데이터가 필요한 컴포넌트 사이의 층수가 줄어듭니다.

## Context 사용 예시

- 테마 지정하기
  - 사용자가 모양을 변경할 수 있는 애플리케이션의 경우 context provider를 앱 최상단에 두고 시각적으로 조정이 필요한 컴포넌트에서 context를 사용할 수 있습니다.
- 현재 계정
  - 로그인한 사용자를 알아야 하는 컴포넌트가 많을 수 있습니다. Context에 놓으면 트리 어디에서나 편하게 알아낼 수 있습니다. 일부 애플리케이션에서는 동시에 여러 계정을 운영할 수도 있습니다. 이런 경우에는 UI의 일부를 서로 다른 현재 계정 값을 가진 provider 로 감싸주는 것이 편리합니다.
- 라우팅
  - 대부분의 라우팅 솔루션은 현재 경로를 유지하기 위해 내부적으로 context를 사용합니다.
- 상태 관리
  - 애플리케이션이 커지면 결국 앱 상단에 수많은 state가 생기게 됩니다. 아래 멀리 떨어진 많은 컴포넌트가 그 값을 변경하고 싶어할 수 있습니다. 흔히 reducer를 context와 함께 사용하는 것은 복잡한 state를 관리하고 번거로운 작업 없이 멀리 있는 컴포넌트까지 값을 전달하는 방법입니다.
