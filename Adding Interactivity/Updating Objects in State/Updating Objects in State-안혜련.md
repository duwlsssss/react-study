# React에서 객체 State 업데이트하기
React에서 state는 불변성을 유지해야 합니다. 
즉, state를 직접 수정하지 않고 항상 새 객체를 생성하여 업데이트해야 합니다. 

### State는 읽기 전용이다.

- state 객체를 직접 변경하면 React가 이를 감지하지 못합니다.
- 변경이 필요하면 새 객체를 생성하여 setState 함수로 전달해야 합니다.
``` jsx
// 잘못된 예
position.x = e.clientX;

// 올바른 예
setPosition({ x: e.clientX, y: e.clientY });
```
### 불변성 유지

- 기존 state를 직접 수정하지 말고, 복사본을 만들어 필요한 값을 변경한 후 다시 설정해야 합니다.

## 객체 복사와 업데이트 방법
###  1. 기본 객체 복사
- 객체 전개 구문(...)을 사용해 객체를 복사합니다.
``` javascript

const [person, setPerson] = useState({ name: 'John', age: 30 });

// 올바른 업데이트 방식
setPerson({
  ...person, // 기존 필드 복사
  age: 31,   // 변경된 값 덮어쓰기
});
```

### 2. 중첩된 객체 복사
- 중첩된 객체를 업데이트하려면 각 단계에서 객체를 복사해야 합니다.
``` javascript
const [person, setPerson] = useState({
  name: 'John',
  details: { city: 'New York', job: 'Developer' },
});

// 올바른 중첩 객체 업데이트
setPerson({
  ...person,
  details: {
    ...person.details,
    city: 'Los Angeles', // 변경된 값 덮어쓰기
  },
});
```
## 중첩된 객체를 더 간편하게 업데이트하는 방법
### Immer를 활용한 간단한 업데이트
Immer는 객체의 불변성을 유지하면서도 간결하게 업데이트할 수 있게 도와주는 라이브러리
``` jsx
// npm install immer use-immer


import { useImmer } from 'use-immer';

const [person, updatePerson] = useImmer({
  name: 'John',
  details: { city: 'New York', job: 'Developer' },
});

// 간결한 업데이트
updatePerson(draft => {
  draft.details.city = 'Los Angeles'; // 직접 변경처럼 작성
});
```

- 중첩된 객체를 간결하게 업데이트할 수 있음.
- 반복적인 전개 구문 작성 없이 불변성을 유지.

## 요약
- React State는 불변성을 유지해야 합니다.

    - 직접 수정하지 말고 항상 새 객체를 생성하여 업데이트하세요.
- 객체 복사하기

    - 얕은 복사: 객체 전개 구문(...)을 사용.
    - 중첩된 객체: 전개 구문을 반복적으로 사용해 단계별로 복사.
- Immer로 간결한 코드 작성
    - Immer를 사용하면 중첩된 객체도 쉽게 업데이트 가능.
    - 복잡한 객체 구조에서 특히 유용.