### 렌더링과 커밋

> 렌더링은 React에서 컴포넌트를 호출하는 것임

React앱의 모든 화면 업데이트는 아래 3단계로 이뤄짐

#### 1. 렌더링 트리거

컴포넌트 렌더링은 `초기 렌더링`, `state 업데이트시`에 일어남

- 초기 렌더링

앱을 시작할 때 React는 루트 컴포넌트를 호출함

```jsx
import React, { StrictMode } from 'react'
import { createRoot } from 'react-dom/client'
import App from './App'

// 대상 DOM 노드와 함께 createRoot를 호출한 다음. 해당 컴포넌트의 render 메서드를 호출
createRoot(document.getElementById('root')).render(
  <StrictMode>
    <App />
  </StrictMode>,
)
```
- state 업데이트시 렌더링(리렌더링)

컴포넌트의 상태를 업데이트하면 자동으로 렌더링 대기열에 추가됨

#### 2. 컴포넌트 렌더링

렌더링을 트리거한 컴포넌트를 호출해 화면에 표시할 내용을 파악함

재귀적 단계를 따름: 업데이트된 컴포넌트가 다른 컴포넌트를 반환하면 해당 컴포넌트를 렌더링하고 해당 컴포넌트도 컴포넌트를 반환하면 반환된 컴포넌트를 다음에 렌더링하는 방식.
중첩된 컴포넌트가 더 이상 없고 React가 화면에 표시되어야 하는 내용을 정확히 알 때까지 이 단계는 계속됨

#### 3. React가 DOM에 변경사항을 커밋

컴포넌트를 렌더링(호출)한 후 마지막으로 React는 DOM을 수정함

- 초기 렌더링의 경우 React는 appendChild() DOM API를 사용해 생성한 모든 DOM 노드를 화면에 표시
- 리렌더링의 경우 React는 필요한 최소한의 작업(렌더링하는 동안 계산된 것)을 적용해 DOM이 최신 렌더링 출력과 일치하도록 함 (렌더링간 차이가 있는 경우에만 DOM 노드를 변경)
