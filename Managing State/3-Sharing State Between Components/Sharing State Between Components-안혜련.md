# Sharing State Between Components

## 1. State 끌어올리기란?

목적: 여러 컴포넌트가 동일한 상태를 공유하고 함께 변경될 필요가 있을 때, 상태를 공통 부모 컴포넌트로 이동하여 관리.

원칙: 상태는 항상 **단일 진리의 원천(Single Source of Truth)**에 존재해야 함.

## 2. State 끌어올리기의 단계

### 1단계: 자식 컴포넌트에서 State 제거
- 자식 컴포넌트는 상태를 직접 관리하지 않고, 부모로부터 전달된 props를 사용.
- 예: Panel 컴포넌트에서 isActive 상태 제거.
``` jsx

function Panel({ title, children, isActive }) {
  return (
    <section className="panel">
      <h3>{title}</h3>
      {isActive ? <p>{children}</p> : <button>Show</button>}
    </section>
  );
}
```

### 2단계: 부모 컴포넌트에서 하드코딩된 State 전달
- 부모 컴포넌트가 자식 컴포넌트의 상태를 하드코딩된 props로 제어.
``` jsx

function Accordion() {
  return (
    <>
      <Panel title="About" isActive={true}>Content 1</Panel>
      <Panel title="Etymology" isActive={false}>Content 2</Panel>
    </>
  );
}
``` 

### 3단계: 부모에 State 추가 및 이벤트 핸들러 전달
 - 부모 컴포넌트에서 상태를 추가로 관리하고, 이벤트 핸들러를 통해 자식 컴포넌트에서 부모 상태를 변경할 수 있도록 구현.

``` jsx

function Accordion() {
  const [activeIndex, setActiveIndex] = useState(0);

  return (
    <>
      <Panel
        title="About"
        isActive={activeIndex === 0}
        onShow={() => setActiveIndex(0)}
      >
        Content 1
      </Panel>
      <Panel
        title="Etymology"
        isActive={activeIndex === 1}
        onShow={() => setActiveIndex(1)}
      >
        Content 2
      </Panel>
    </>
  );
}

function Panel({ title, children, isActive, onShow }) {
  return (
    <section className="panel">
      <h3>{title}</h3>
      {isActive ? (
        <p>{children}</p>
      ) : (
        <button onClick={onShow}>Show</button>
      )}
    </section>
  );
}
```
