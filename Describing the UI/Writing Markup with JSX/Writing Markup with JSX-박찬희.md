# Writing Markup with JSX

JSX는 JavaScript를 확장한 문법으로 JavaScript 파일을 HTML과 비슷하게 마크업을 작성 할 수 있도록 해주십니다. 컴포넌트를 작성하는 방법도 있지만, 대부분의 React 개발자는 JSX의 간결함을 선호하며 대부분의 코드 베이스에서 JSX를 사용합니다.

## JSX:JavaScript에 마크업 넣기

Web은 HTML,CSS,JavaScript를 기반으로 만들어져 왔습니다. 수년 동안 웹 개발자는 HTML,CSS,JavaScript로 각각 문서 내용, 디자인, 로직을 분리된 파일로 관리하며 작성해 왔습니다.

Web이 시간이 지나 더욱 인터렉티브해지면서, 로직이 내용을 결정하는 경우가 많아졌습니다. HTML을 JavaScript에서 담당하는 경우가 생긴 것입니다. 그렇기 때문에 React는 컴포넌트에서 랜더링 로직과 마크업이 같은 위치에 함께 있게 되었습니다.

```jsx
//Sidebar.js React component

Sidebar(){
  if(isLoggedIn()){
    <p>Welcome</p>
  }else{
    <Form />
  }
}
```
```jsx
//Form.js React component

Form(){
  onClick(){...}
  onSubmit(){...}
  <form onSubmit>
    <input onClick/>
    <input onClick/>
  </form>
}
```
버튼의 랜더링 로직과 버튼의 마크업이 함께 있으면, 매번 변화가 생길 때 마다 서로 동기화 상태를 유지할 수 있습니다. 반대로 버튼의 마크업과 사이바의 마크업처럼 서로 관련이 없는 항목들은 서로 분리되어 있으므로, 각각 개별적으로 변경하는 것이 더 안전합니다.

각 React 컴포넌트는 React가 브라우저에 마크업을 랜더링 할 수 있는 JavaScript함수입니다. React 컴포넌트는 JSX라는 확장된 문법을 사용하여 마크업을 나타냅니다. JSX는 HTML과 비슷해보이지만, 조금 더 엄격하며 동적으로 정보를 표시할 수 있습니다. JSX를 이해하는 가장 좋은 방법은 HTML마크업을 JSX 마크업으로 변환해 보는 것입니다.

**JSX와 React는 사로 다른 별개의 개념으로 함께 사용되기도 하지만 독립적으로 사용할 수 있습니다. JSX는 확장된 문법이고, React는 JavaScript 라이브러리 입니다.**

## HTML을 JSX로 변환하기
```html
<h1>Hedy Lamarr's Todos</h1>
<img
  src="https://i.imgur.com/yXOvdOSs.jpg"
  alt="Hedy Lamarr"
  class="photo"
>
<ul>
    <li>Invent new traffic lights
    <li>Rehearse a movie scene
    <li>Improve the spectrum technology
</ul>
```
다음 HTML을 컴포넌트로 만들어 보겠습니다.(오류가 없는 코드라 가정합니다.)

```jsx
// /src/App.js: Adjacent JSX elements must be wrapped in an enclosing tag. Did you want a JSX fragment <>...</>? (5:4) 마크업을 수정하라는 오류메시지가 출력됩니다.
export default function TodoList() {
  return (
    <h1>Hedy Lamarr's Todos</h1>
    <img
      src="https://i.imgur.com/yXOvdOSs.jpg"
      alt="Hedy Lamarr"
      class="photo"
    >
    <ul>
        <li>Invent new traffic lights
        <li>Rehearse a movie scene
        <li>Improve the spectrum technology
    </ul>
  )
}
```
이렇게 코드를 그대로 복사할 경우 동작하지 않습니다. JSX는 HTML보다 더 엄격하고 몇 가지 규칙이 더 있기 때문입니다. 오류 메시지를 읽으면 마크업을 수정하도록 안내하고 있습니다.

## JSX의 규칙
### 1. 하나의 루트 엘리먼트로 반환하기
한 컴포넌트에서 여러 엘리먼트를 반환하려면, **하나의 부모 태그로 감싸줘야합니다.**
예를 들면 `<div>` 또는 `<></>`를 사용할 수 있습니다.
```jsx
//div 대신 빈태그(<></>)를 사용할 수 있습니다
<div>
  <h1>Hedy Lamarr's Todos</h1>
  <img
    src="https://i.imgur.com/yXOvdOSs.jpg"
    alt="Hedy Lamarr"
    class="photo"
  >
  <ul>
    ...
  </ul>
</div>
```
빈 태그를 Fragment라고 합니다. Fragments는 브라우저상의 HTML트리 구조에서 흔적을 남기지 않고 그룹화해줍니다.
#### JSX에서 태그로 감싸줘야하는 이유
JSX는 HTML처럼 보이지만 내부적으로는 일반 JavaScript 객체로 변환됩니다. 하나의 배열로 감싸지 않은 하나의 함수에서는 두 개의 객체를 반환할 수 없습니다. 그렇기 때문에 다른 태그나 Fragment로 감싸지 않으면 두 개의 JSX태그를 반환하는 것이기 때문에 반환되지 않습니다.

### 2. 모든 태그는 닫아주기
JSX에서는 태그를 명시적으로 닫아야 합니다. `<img>`처럼 자체적으로 닫아주는 태그는 반드시 `<img />`형태로 작성해야 하며, `<li>oranges`와 같은 래핑 태그도 `<li>oranges</li>` 형태로 작성해야 합니다.

수정하면 다음과 같이 이미지와 리스트 항목들은 닫아줍니다.
```jsx
<>
  <img
    src="https://i.imgur.com/yXOvdOSs.jpg"
    alt="Hedy Lamarr"
    class="photo"
   />
  <ul>
    <li>Invent new traffic lights</li>
    <li>Rehearse a movie scene</li>
    <li>Improve the spectrum technology</li>
  </ul>
</>
```
### 3. 대부분 카멜 케이스로(camelCase)
JSX는 Javascript로 바뀌고 JSX에서 작성된 어트리뷰트는 JavaScript 객체의 키가 됩니다. 컴포넌트에서는 종종 어트리뷰트를 변수로 읽고 싶은 경우가 있습니다. 그러나 JavaScript는 변수명에 제한이 있습니다. 예를 들어 변수명에 대시(_)를 포함하거나 `class`처럼 예약어를 사용할 수 없습니다.

이것이 바로 React에서 HTML과 SVG의 어트리뷰트 대부분이 카멜 케이스로 작성되는 이유입니다. 예를 들면, `stroke-width` 대신 `strokeWidth`로 사용합니다. `class`는 예약어이기 때문에, React에서는 DOM의 프로퍼티의 이름을 따서 `className`으로 대신 작성합니다.

```jsx
<img
  src="https://i.imgur.com/yXOvdOSs.jpg"
  alt="Hedy Lamarr"
  className="photo"
/>
```
**역사적인 이유로, aria-*와 data-*의 어트리뷰트는 HTML에서와 동일하게 대시를 사용하여 작성합니다.**

### 팁: JSX 변환기 사용하기 
기존 마크업에서 모든 어트리뷰트를 변환하는 것은 지루할 수 있습니다. 변환기를 사용하여 기존 HTML과 SVG를 JSX로 변환하는 것을 추천합니다. 변환기는 매우 유용하지만 그래도 JSX를 혼자서 편안하게 작성할 수 있도록 어트리뷰트를 어떻게 쓰는지 이해하는 것도 중요합니다.

최종 코드는 다음과 같습니다.
```jsx
export default function TodoList() {
  return (
    <>
      <h1>Hedy Lamarr's Todos</h1>
      <img
        src="https://i.imgur.com/yXOvdOSs.jpg"
        alt="Hedy Lamarr"
        className="photo"
      />
      <ul>
        <li>Invent new traffic lights</li>
        <li>Rehearse a movie scene</li>
        <li>Improve the spectrum technology</li>
      </ul>
    </>
  );
}
```