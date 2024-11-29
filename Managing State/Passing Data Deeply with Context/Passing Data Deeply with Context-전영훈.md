# Context를 사용해 데이터를 깊게 전달하기

보통의 경우 부모 컴포넌트에서 자식 컴포넌트로 props를 통해 정보를 전달합니다. 그러나 부모에서 원하는 자식 컴포넌트까지 도달하는데 까지 여러 컴포넌트 층을 거치면, 흔히말하는 ugly 코드가 되어버립니다.

=> 이때 Context는 부모 컴포넌트가 트리 아래에 있는 모든 컴포넌트에 깊이에 상관없이 정보를 명시적으로 props를 통해 전달하지 않고도 사용할 수 있게 해줍니다.

## ”Prop drilling” 이란?

부모 컴포넌트에서 원하는 자식 컴포넌트까지 props로 데이터를 쭉쭉 전달하는것

## Context: Props 전달하기의 대안

### 1단계: context생성하기

```jsx
import { createContext } from "react";

export const LevelContext = createContext(1);
```

=>createContext의 유일한 인자는 기본값입니다.

### 2단계: Context 사용하기

=> 기존에 반복되었던 props값을 없애고 그 상위요소가 대신 값을 받도록 업데이트합니다

### 3단계: Context 제공하기

```jsx
import { LevelContext } from "./LevelContext.js";

export default function Section({ level, children }) {
  return (
    <section className="section">
      <LevelContext.Provider value={level}>{children}</LevelContext.Provider>
    </section>
  );
}
```

상위요소( 여기선 section)의 children을 context provider로 감싸주면, 리액트가 Section 내의 어떤 컴포넌트가 LevelContext를 요구하면 level을 주라고 알려줍니다.

정리하자면,

1. level prop 을 <Section>에 전달합니다.
2. Section은 자식을 <LevelContext.Provider value={level}>로 감싸줍니다.
3. Heading은 useContext(LevelContext)를 사용해 가장 근처의 LevelContext의 값을 요청합니다.

## 만약 같은 컴포넌트에서 context를 사용하며 제공하길 원한다면?

```jsx
   <Section level={1}>
      ...
      <Section level={2}>
        ...
        <Section level={3}>
```

다음과 같이 수동적으로 지정할 수도 있으나,

```jsx
<section className="section">
  <LevelContext.Provider value={level + 1}>{children}</LevelContext.Provider>
</section>
```

context provider에 value를 지정해줌으로써 각 Section은 위의 Section에서 level을 읽고 자동으로 level + 1을 아래로 전달할 수 있습니다

## Context를 사용하기 전에 고려할 것

1. 일단 Props로 전달 시작하기. 컴포넌트 구조가 간단하다면, props로 전달하는 것이 데이터 흐름을 명확하게 하기 좋습니다
2. 컴포넌트를 추출하고 jsx를 children값으로 전달하기.
   <Layout><Posts posts={posts} /><Layout>
   이런 코드로 작성해야 그나마 부모값에 더 가까워져 props전달 층수가 줄어듭니다.
