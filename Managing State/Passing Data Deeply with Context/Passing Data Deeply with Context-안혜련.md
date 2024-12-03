# Context를 사용해 데이터를 깊게 전달하기

보통의 경우 부모 컴포넌트에서 자식 컴포넌트로 props를 통해 정보를 전달한다.

그러나 중간에 많은 컴포넌트를 거쳐야 하거나, 애플리케이션의 많은 컴포넌트에서 동일한 정보가 필요한 경우에는 props를 전달하는 것이 번거롭고 불편할 수 있다.

Context는 부모 컴포넌트가 트리 아래에 있는 모든 컴포넌트에 깊이에 상관없이 정보를 명시적으로 props를 통해 전달하지 않고도 사용할 수 있게 해준다

## Prop Drilling

- 데이터를 트리의 루트-> 하위 컴포넌트까지 전달하는 방식.
- 중간 컴포넌트가 데이터를 단순히 전달만 할 때 비효율적.

```jsx
function App() {
  const data = "Hello";
  return <Parent data={data} />;
}

function Parent({ data }) {
  return <Child data={data} />;
}

function Child({ data }) {
  return <div>{data}</div>;
}
```

중간의 Parent 컴포넌트는 데이터를 사용하지 않고 전달만 하므로 불필요한 코드가 증가.

### Context의 대안으로서의 역할:

- Context는 부모 컴포넌트가 데이터를 트리 아래로 전달할 때 중간 단계를 생략.
- 트리의 어디서든 필요하면 데이터를 직접 사용할 수 있게 함.

## Context 사용 방법

### 1 . Context 생성

```jsx
import { createContext } from "react";

export const MyContext = createContext(defaultValue);
```

- defaultValue: Context를 제공하지 않을 때 사용되는 기본값.

### 2. Context 제공

부모 컴포넌트에서 데이터를 제공하려면 `<Provider>`를 사용.

```jsx
import { MyContext } from "./MyContext";

function Parent() {
  return (
    <MyContext.Provider value="Hello from Context">
      <Child />
    </MyContext.Provider>
  );
}
```

### 3. Context 사용

- 자식 컴포넌트에서 useContext 훅으로 데이터를 읽음.

```jsx
import { useContext } from "react";
import { MyContext } from "./MyContext";

function Child() {
  const value = useContext(MyContext);
  return <div>{value}</div>;
}
```

## 예를 들어

`<Heading>` 컴포넌트가 제목의 레벨(예: `<h1>`, `<h2>`)을 결정해야 할 때, Context를 활용해 부모 섹션의 레벨에 따라 자동으로 설정.

Context 생성:

```jsx
import { createContext } from "react";
export const LevelContext = createContext(1); // 기본값: 1
```

```jsx
import { LevelContext } from "./LevelContext";

function Section({ level, children }) {
  return (
    <LevelContext.Provider value={level}>
      <section>{children}</section>
    </LevelContext.Provider>
  );
}
```

어떻게 하냐면

```jsx
import { useContext } from "react";
import { LevelContext } from "./LevelContext";

function Heading({ children }) {
  const level = useContext(LevelContext);
  switch (level) {
    case 1:
      return <h1>{children}</h1>;
    case 2:
      return <h2>{children}</h2>;
    case 3:
      return <h3>{children}</h3>;
    default:
      throw new Error("Unsupported level");
  }
}
```

## Context의 일반적인 사용 사례

- 테마 관리:
  - 다크 모드와 같은 테마를 트리 전역에서 관리.
- 사용자 정보:
  - 로그인한 사용자 정보 공유.
- 라우팅:
  - 현재 경로나 활성 링크 상태 전달.
- 상태 관리:
  - 전역 상태를 관리하기 위해 reducer와 함께 사용.

## Context 사용 시 고려사항

- Context를 남용하지 말 것:

  - 모든 데이터를 Context에 넣으면 데이터 흐름을 파악하기 어렵고 성능 저하가 발생할 수 있음.

- 단순한 구조에서는 props 전달이 더 명확하고 유지보수에 적합.
- 중간 컴포넌트가 데이터를 단순히 전달만 한다면 컴포넌트를 재구성하거나 JSX의 children 속성을 활용.
