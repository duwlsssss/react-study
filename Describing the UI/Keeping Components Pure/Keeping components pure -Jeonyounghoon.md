# 순수함수

순수 함수는 오직 연산만을 수행하는 함수를 말합니다

## 순수함수 왜 쓰나?

1. 입력이 변경되지 않은 컴포넌트 렌더링을 건너뛰어 성능을 향상시킬 수 있습니다.
2. 순수 함수는 항상 동일한 결과를 반환하므로 캐시하기에 안전합니다.

React는 이러한 컨셉을 기반으로 설계되었습니다. React는 작성되는 모든 컴포넌트가 순수 함수일 거라 가정합니다. 이러한 가정은 작성되는 React 컴포넌트에 같은 입력이 주어진다면 반드시 같은 JSX를 반환한다는 것을 의미합니다.

다음은 react의 순수 함수 사용 예제입니다

```jsx
function Cup({ guest }) {
  return <h2>Tea cup for guest #{guest}</h2>;
}

export default function TeaSet() {
  return (
    <>
      <Cup guest={1} />
      <Cup guest={2} />
      <Cup guest={3} />
    </>
  );
}
```

=> guest props를 통해 일정한 값이 독립적으로 전달되어 연산됨
