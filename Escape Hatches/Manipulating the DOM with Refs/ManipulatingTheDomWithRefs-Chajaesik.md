### 1. 특정 노드에 포커스 옮기기

### 2. 스크롤 위치 옮기기

### 3. 위치와 크기를 측정

우리는 위와 같은 상황 등에서 React가 관리하는 DOM 요소에 접근해야 할 때가 있다.

React는 이런 작업을 수행하는 내장 방법을 제공하지 않기 때문에 DOM 노드에 접근하기 위한 ref가 필요하다.

### 학습 내용
- ref 어트리뷰트로 React가 관리하는 DOM 노드에 접근하는 법
- ref JSX 어트리뷰트와 useRef Hook의 관련성
- 다른 컴포넌트의 DOM 노드에 접근하는 법
- React가 관리하는 DOM을 수정해도 안전한 경우
### ref로 노드 가져오기
먼저 React가 관리하는 DOM 노드에 접근하기 위해 useRef Hook을 가져온다.

`import { useRef } from 'react';`
컴포넌트 안에서 ref를 선언하기 위해 방금 가져온 Hook을 사용한다.

`const myRef = useRef(null);`
마지막으로 ref를 DOM 노드를 가져와야하는 JSX tag에 ref 어트리뷰트로 전달한다.

`<div ref = {myRef}`
 

이전 시간에 우리는 useRef Hook은 current라는 단일 속성을 가진 객체를 반환한다고 배웠다.

초기에는 myRef.current가 null이 된다.

React가 이 <div>에 대한 DOM 노드를 생성할 때, React는 이 노드에 대한 참조를 myRef.current에 넣는다.

```
// 예를 들어 이렇게 브라우저 API를 사용할 수 있다.
myRef.current.scrollIntoView();
 ```

### 예시 : 텍스트 입력에 포커스 이동하기
```
export default function Form() {
	const inputRef = useRef(null); // useRef Hook을 사용
    
    function handleClick() {
        // 1. input.current에서 input DOM 노드를 읽고
        // 2. inputRef.current.focus()로 focus()를 호출
		inputRef.current.focus(); 
   	}
    
    return(
    	<>  // React에게 이 <input>의 DOM 노드를 inputRef.current에 넣어줘
        	<input ref = {inputRef} />
            <button onClick = {handleClick}> // button의 onClick으로 handleCLick 이벤트헨들러 전달
            	Focus the input
            </button>
        </>
      );
   }
 ```

### 예시 : 한 요소로 스크롤 이동하기
한 컴포넌트에서 하나 이상의 ref를 가질 수 있다.

아래의 각 버튼은 브라우저 scrollIntoView() 메서드를 해당 DOM 노드로 호출하여 이미지를 중앙에 배치한다.
```
export default function CatFriends() {
	const firstCatRef = useRef(null);
    const secondCatRef = useRef(null);
    const thirdCatRef = useRef(null);
    
    function handleScrollToFirstCat(){
    	firstCatRef.current.scrollIntoView({
        	behavior: 'smooth',
            block:	'nearest',
            inline: 'center'
        });
    }
    
    function handleScrollToSecondCat(){
    	secondCatRef.crreunt.scrollIntoView({
        	...
            
            <img
              src="https://placecats.com/neo/300/200"
              alt="Neo"
              ref={firstCatRef}
            />
            ..
 ```

### 다른 컴포넌트의 DOM 노드 접근하기
`<input />` 같은 브라우저 요소를 출력하는 내장 컴포넌트에 ref를 주입할 때 React는 ref의 current 프로퍼티를 그에 해당하는 (브라우저의 실제 `<input />` 같은} DOM 노드로 설정한다.

 

하지만 `<MyInput />` 같이 직접 만든 컴포넌트에 ref를 주입할 때는 null이 기본적으로 주어진다.

 

이를 통해 React는 기본적으로 다른 컴포넌트의 DOM 노드에 접근하는 것을 허용하지 않는다.

대신 특정 컴포넌트에서 소유한 DOM 노드를 선택적으로 노출할 수 있다.

컴포넌트는 자식 중 하나에 ref를 "전달"할 수 있다.

 

### 아래는 MyInput이 forwardRef API를 사용할 수 있는지 살펴 볼 수 있다.
```
const MyInput = forwardRef((props, ref) => {
	return <input { ...props} ref={ref} /> ;
});
```
### 작동하는 원리는 다음과 같다.

- <MyInput ref={inputRef} />으로 React가 대응되는 DOM 노드를 inputRef.current에 대입하도록 설장한다. 하지만 기본적으로 이렇게 동작하진 않는다.
- MyInput 컴포넌트는 forwardRef를 통해 선언되었다. 이것은 props 다음에 선언된 두번째 ref 인수를 통해 상위의 inputRef를 받을 수 있게 한다.
- Myinput은 자체적으로 수신받은 ref를 컴포넌트 내부의 `<input>`으로 전달한다.
```
import { forwardRef, useRef } from 'react'

const MyInput = forwardRef((props, ref) => {
	return <input {...props} ref = {ref} />;
 });
 
 export default function Form() {
 	const inputRef = useRef(null);
    
    function handleClick(){
    	inputRef.current.focus();
    }
    
    return (
    <>
    	<MyInput ref={inputRef} />
        <button onClick = {handleCLick}>
        	Focus the input
        </button>
    </>
  );
}
 
```

### React가  ref를 부여할 때
React의 모든 갱신은 두 단계로 나눌 수 있다.

- 렌더링 단계에서 React는 화면에 무엇을 그려야 하는지 알아내도록 컴포넌트를 호출한다.
- 커밋 단계에서 React는변경사항을 DOM에 적용한다.

첫 렌더링에서 DOM 노드는 아직 생성되지 않아 ref.current는 null인 상태이다.

그리고 갱신에 의한 렌더링에서 DOM 노드는 아직 업데이트되지 않은 상태이다.

위의 두 상황 모두  ref를 읽기에 너무 이른 상황이다.

 

React는 ref.current를 커밋 단계에서 설정한다.

DOM을 변경하기 전에 React는 관련된 ref.current 값을 미리 null로 설장한다.

그리고 DOM을 변경한 후 React는 즉시 대응하는 DOM 노드로 다시 설정한다.

 

대부분의 ref 접근은 이벤트 핸들러 안에서 일어난다.

 

### React가 관리하는 DOM 노드를 직접 바꾸려 하지마!

안전하게 React가 업데이트 할 이유가 없는 DOM 노드 일부를 수정할 수는 있다.