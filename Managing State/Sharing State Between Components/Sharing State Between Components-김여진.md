### 컴포넌트 간 State 공유하기

두 컴포넌트의 state가 항상 함께 변경되길 원하면
각 컴포넌트에서 state를 제거하고 부모 컴포넌트에 state를 추가, 
자식에서 부모의 state를 반영,변경할 수 있도록 상태와 이벤트핸들러를 props로 전달해야함 : lifting state up

상태 변경의 책임이 중앙에 집중됨

React에서 단방향 데이터 흐름과 컴포넌트 간 책임 분리 구현하는 패턴임

```jsx
//아코디언 컴포넌트에서 자식 패널이 하나씩만 활성화되게 하고 싶음
export default function Accordion() {
  const [activeIndex, setActiveIndex] = useState(0); // 부모에서 활성화된 자식 Panel의 인덱스 사용

  return (
    <>
      <Panel 
        title="제목1"
        isActive={activeIndex === 0}
        onShow={() => setActiveIndex(0)}
      >
        내용1
      </Panel>
      <Panel title="제목2" 
        isActive={activeIndex === 1}
        onShow={() => setActiveIndex(1)}
      >
        내용2
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
        <button onClick={onShow}> {/*자식 컴포넌트로 이벤트 핸들러를 넘겨 자식에서 부모의 상태를 변경하게 함*/}
          Show
        </button>
      )}
    </section>
  );
}
```

여기서 isActive prop을 갖는 Panel 컴포넌트는 Accordion 컴포넌트에 의해 제어됨. 
즉 부모 컴포넌트(제어 컴포넌트)가 동작을 완전히 지정하고 자식 컴포넌트(비제어 컴포넌트)는 부모 컴포넌트로부터 제어받음

컴포넌트 작성시 props를 통해 다른 컴포넌트를 `제어해야`하는지, state를 통해 `제어받아야`하는지 고려해 설계하자
