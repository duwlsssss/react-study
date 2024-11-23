### state 업데이트

```jsx
import { useState } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0); 

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => {
        setCount(count + 1) {/* count는 0, +1  요청 */}
        setCount(count + 1) {/*여전히 count는 0으로 참조, +1 요청 */}
        setCount(count + 3) {/*여전히 count는 0으로 참조, +3 요청 -> 최종 count는 3만큼 증가*/}
      }}>Increment</button>
    </div>
  );
}
// 버튼을 클릭해 컴포넌트가 리렌더링될때마다 state는 해당 렌더링에 고정됨
// 즉 렌더링 도중 count를 여러번 변경하려 해도 동일한 값을 참조함
// React는 모든 이벤트 핸들러 실행을 마친 후 state 업데이트를 batch 처리함 (같은 렌더링 중 여러 번의 setState 호출을 수집한 후 처리)


// ➕ state가 이전값을 기반으로 업데이트 돼야할 땐  업데이터 함수 사용하기
// 업데이터 함수는 렌더링 중에 실행돼며 이전 업데이트 함수의 반환값. 즉 가장 최종 업데이트된 state 값을 참조해 사용
// 업데이터 함수는 렌더링 중에 실행되므로 순수함수여야함
<button onClick={() => {
  setCount(c => c + 1); {/*첫 번째 호출: 이전 값(c) + 1*/}
  setCount(c => c + 1); {/*두 번째 호출: 첫 번째 호출 이후 값(c) + 1*/}
  setCount(c => c + 3); {/*세 번째 호출: 두 번째 호출 이후 값(c) + 3 -> 최종 count는 5만큼 증가*/}
}}>
  Increment
</button>
```