# State: A Component's Memory
컴포넌트는 상호작용의 결과로 화면의 내용을 변경해야 하는 경우가 많다. 

폼에 입력한 값, 캐러셀의 다음 이미지, 장바구니에 추가된 아이템 등을 기억해야 하는데, 이런 종류의 컴포넌트별 메모리를 **state(상태)**라고 한다.

## 컴포넌트를 새로운 데이터로 업데이트하기 위해서

1. 렌더링 사이에 데이터를 **유지**한다. 
2. React가 새로운 데이터로 컴포넌트를 렌더링하도록 **유발**한다.


## useState
이 훅은 두가지를 제공한다.

1. 렌더링 간에 데이터를 유지하기 위한 **state 변수**.
2. 변수를 업데이트하고 React가 컴포넌트를 다시 렌더링하도록 유발하는 **state setter 함수**

## useState 해부하기

`useState`를 호출하는 것은, React에 이 컴포넌트가 무언가를 기억하기를 원한다고 말하는 것이다.
``` jsx
const [index, setIndex] = useState(0);
```
이 경우 React가 `index`를 기억하기를 원한다.

컴포넌트가 렌더링될 때마다, `useState`는 다음 두 개의 값을 포함하는 배열을 제공한다.

1. 저장한 값을 가진 **state 변수** (`index`).
2. state 변수를 업데이트하고 React에 컴포넌트를 다시 렌더링하도록 유발하는 **state setter 함수** (`setIndex`).

## state

- State는 화면에서 컴포넌트 인스턴스에 지역적입니다. 
- **동일한 컴포넌트를 두 번 렌더링한다면 각 복사본은 완전히 격리된 state를 가집니다!** 그중 하나를 변경해도 다른 하나에는 영향을 미치지 않습니다.
- state는 컴포넌트에 비공개입니다. 두 곳에서 렌더링하더라도 각각의 복사본은 고유한 state를 가집니다.