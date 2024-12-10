### Ref로 DOM 조작하기

리액트는 렌더링 결과에 맞춰 DOM을 자동으로 변경함. 하지만 특정 DOM element에 직접 접근해야 하는 상황도 있음
이때 ref를 DOM 노드를 가져와야하는 JSX tag 에 `ref 어트리뷰트`로 전달
그럼 current 프로퍼티에 해당 DOM 노드에 대한 참조가 들어감 -> 이벤트 핸들러에서 조작, 노드에 저장된 내장 브라우저 API 사용 가능

```js
// 버튼 클릭시 해당 요소로 스크롤 이동
import { useRef, useState } from "react";

export default function CatFriends() {
  const itemsRef = useRef(new Map());
  const [catImages, setCatImages] = useState(setupCatList);

  function scrollToCat(cat) { // 2. 버튼 클릭시
    const node = itemsRef.current.get(cat); // 특정 cat url에 매핑된 DOM 노드를 가져옴
    node?.scrollIntoView({ // 내장 브라우저 API 사용
      behavior: "smooth",
      block: "nearest",
      inline: "center",
    });
  }

  return (
    <>
      <nav>
        {catImages.map((cat, i) => (
           <button key={i} onClick={() => scrollToCat(cat)}>
            Scroll to Cat {i + 1}
           </button>
        ))}
      </nav>
      <div>
        <ul>
          {catList.map((cat) => (
            <li
              key={cat}
              ref={(node) => { // 1.<li> DOM 노드 생성시 ref 콜백이 호출되어 DOM 노드가 itemsRef.current의 Map에 저장됨
                if (node) {
                  itemsRef.current.set(cat, node);
                } else {
                  itemsRef.current.delete(cat); // 컴포넌트 언마운트, 해당 DOM요소가 렌더링에서 제거될떄  Map에서 DOM 노드 제거
                }
              }}
            >
              <img src={cat} alt={`Cat ${cat}`}/>
            </li>
          ))}
        </ul>
      </div>
    </>
  );
}

function setupCatList() {
  const catList = [];
  for (let i = 0; i < 10; i++) {
    catList.push("https://loremflickr.com/320/240/cat?lock=" + i);
  }

  return catList;
}
```

<input /> 같은 브라우저 요소를 출력하는 내장 컴포넌트의 경우 ref 어트리뷰트 사용시 
React는 ref의 current 프로퍼티를 그에 해당하는 DOM 노드로 설정함

하지만 커스텀 컴포넌트에 ref 어트리뷰트를 사용하면 기본적으로 `null`이 들어감

-> 사용자 정의 컴포넌트에서 DOM 노드값을 사용하려면 `forwardRef` 사용해 컴포넌트를 선언 (부모 컴포넌트에서 자식 컴포넌트의 DOM 노드에 ref를 전달할 수 있게 해주는 React 기능)

```js
import { forwardRef, useRef } from 'react';

// ref를 두번째 매개변수로 받아 특정 DOM 노드에 연결
const MyInput = forwardRef((props, ref) => {
  return <input {...props} ref={ref} />;
});

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>
        Focus the input
      </button>
    </>
  );
}
```

⚠️ Ref로 React가 관리하는 DOM 노드를 직접 조작하면 React가 만들어내는 변경 사항과 충돌을 발생 위험 -> React가 관리하는 DOM 노드를 수정하려 한다면, React가 변경할 이유가 없는 부분만 수정하기 

```js
function App() {
  const divRef = useRef(null);

  function handleDirectManipulation() {
    divRef.current.textContent = "Directly Updated!"; // 직접 조작
  }

  return (
    <div>
      <div ref={divRef}>Hello, React!</div> {/* 이후 React가 해당 <div>를 리렌더링할 경우, React는 가상 DOM 상태("Hello, React!") 로 DOM을 덮어씌워 충돌이 발생 */}
      <button onClick={handleDirectManipulation}>Direct DOM Update</button>
    </div>
  );
}


// React가 관리하는 DOM 노드를 수정하려 한다면, React가 변경할 이유가 없는 부분만 수정하기
function App() {
  const divRef = useRef(null);

  function handleDirectStyleChange() {
    divRef.current.style.backgroundColor = "red"; // React는 backgroundColor를 관리하지 않음, React가 이를 덮어쓰지 않음
  }

  return (
    <div>
      <div ref={divRef} style={{ color: "blue" }}>Hello, React!</div>
      <button onClick={handleDirectStyleChange}>Change Background</button>
    </div>
  );
}
```