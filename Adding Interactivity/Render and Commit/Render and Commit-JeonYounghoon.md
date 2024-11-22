# 렌더링 그리고 커밋

리액트 렌더링 동작에 대한 이해

## 리액트의 UI렌더링 동작 단계

리액트 렌더링은 1.렌더링 트리거, 2.컴포넌트 렌더링, 3.DOM에 커밋 순서대로 일어납니다

## 1. 렌더링 트리거 (주문전달)

컴포넌트 렌더링은 초기 렌더링, state 업데이트시에 일어납니다

### 초기렌더링

import Image from './Image.js';
import { createRoot } from 'react-dom/client';

const root = createRoot(document.getElementById('root'))
root.render(<Image />);

앱을 시작할 때, root를 호출후 render 메서드를 호출함으로서 초기렌더링이 일어납니다.

### State로 렌더링 트리거

State 없데이트시 리렌더링 state의 set함수를 통하여 추가적인 렌더링을 트리거할 수 있습니다.

## 2. 컴포넌트 렌더링 (주문 준비)

렌더링을 트리거한 후 React는 컴포넌트를 호출하여 화면에 표시할 내용을 파악하는 것을 말합니다

업데이트된 컴포넌트가 다른 컴포넌트를 반환하면 React는 해당 컴포넌트를 렌더링하고 해당 컴포넌트도 컴포넌트를 반환하면 반환된 컴포넌트를 다음에 렌더링하는 방식의 재귀적 방법을 따른다고 합니다.

## 3. DOM에 커밋 (요리 내놓기)

컴포넌트를 렌더링 (JSX를 Virtual DOM 형태로 변환)한 후 React는 Virtual DOM에서 계산된 변경 사항을 토대로 실제 DOM을 수정합니다.

- 초기 렌더링의 경우 React는 appendChild() DOM API를 사용하여 생성한 모든 DOM 노드를 화면에 표시합니다.
- 리렌더링의 경우 DOM이 state에서 업데이트된 데이터를 가지고 최신 렌더링 출력과 일치하도록 합니다
