
## 1. Effect와 컴포넌트 생명주기의 차이
- 컴포넌트 생명주기: 마운트 → 업데이트 → 마운트 해제.
- Effect 생명주기: 동기화 시작 → 동기화 중지.
- Effect는 컴포넌트 생명주기와 독립적으로 동작하며, props나 state의 변경에 따라 여러 번 시작/중지될 수 있음.
## 2. Effect의 핵심 구조
- 동기화 시작 코드:
``` jsx
const connection = createConnection(serverUrl, roomId);
connection.connect();
```
- 동기화 중지(cleanup) 코드:
``` jsx

return () => {
  connection.disconnect();
};
```

React는 effect를 실행하거나 cleanup 함수를 호출하며 동기화를 제어.

## 3. Effect의 의존성(dependencies)
- Effect가 의존하는 값(props, state 등)이 변경되면 React는 effect를 다시 실행.
- 의존성 배열([])의 역할:
    - 배열에 명시된 값이 변경되었을 때만 effect가 다시 실행됨.
    - 빈 배열([]): 마운트 시 한 번 실행, 마운트 해제 시 cleanup 실행.
## 4. Effect가 다시 동기화되는 이유
- props/state 변화:
    - 예: roomId가 "general"에서 "travel"로 변경 → 이전 방에서 연결 해제 후 새 방에 연결.
- 효율적인 재동기화:
    - React는 필요한 경우만 effect를 다시 실행해 불필요한 작업을 줄임.
## 5. Effect를 작성하는 팁
- 각 effect는 독립적인 동기화 프로세스를 나타냄:
    - 서로 다른 목적의 동기화를 하나의 effect에 합치지 말고 분리.
    - 예: 서버 연결과 분석 로직은 별도의 effect로 작성.
- React의 의존성 규칙 준수:
    - Effect 내부에서 읽은 모든 반응형 값은 의존성 배열에 포함.
    - 린터는 누락된 의존성을 지적하므로 오류를 무시하지 말 것.
## 6. 반응형 값(Reactive Values)
- 반응형 값: props, state, 그리고 렌더링 중 계산된 값.
    - 변경될 가능성이 있으므로 항상 의존성 배열에 포함.
- 반응형이 아닌 값:
    - 컴포넌트 외부에서 선언되거나 상수로 정의된 값은 의존성 배열에 포함하지 않아도 됨.
## 7. React 린터와 종속성
- Effect 내부에서 읽은 반응형 값이 의존성 배열에 포함되었는지 확인.
- 종속성 관련 주요 규칙:
    - 의존성 누락 시: Effect가 오래된 값을 참조할 수 있는 버그 발생.
    - 잘못된 종속성 추가 시: 불필요한 반복 실행 발생.
## 8. Effect 관련 흔한 실수와 해결법
- 무한 루프 발생:
    - 의존성 배열에 불필요한 값을 포함한 경우.
    - 해결: effect 내부의 반응형 값만 포함.
- 린터 무시:
    - // eslint-disable-next-line로 린터를 억제하는 것은 문제를 숨기는 것에 불과.
    - 해결: 값이 반응형인지 확인하고, 종속성을 명확히 설정.
## 9. 효과적으로 Effect를 작성하는 방법
- 의존성 정의:
    - 필요한 값을 모두 포함하고, 필요 없는 값은 제외.
- 의존성 값 줄이기:
    - 반응형 값이 아닌 값을 컴포넌트 외부로 이동.
- Example:
``` jsx

const serverUrl = 'https://localhost:1234'; // 반응형 값 아님
```
## 10. React가 효과적으로 Effect를 다시 동기화하는 이유
- Effect 내부에서 반응형 값을 읽고, 값이 변경되면 다시 실행하여 동기화 상태를 유지.
- 컴포넌트 생명주기를 고려하지 않고 개별 effect가 필요한 동작을 수행.
