# State를 사용해 Input 다루기

## 1. 선언형 UI란?

- 명령형 UI: UI를 직접 조작하며, 상태 변화에 따라 버튼 활성화, 숨김/보임 등을 명령(hide, show)으로 구현.

  - 문제점: 복잡성이 증가하면 버그 발생 가능성 상승.
  - 운전기사에게 한 번 한 번의 턴을 알려주는 방식.

- 선언형 UI: 상태(state)에 따라 UI가 어떤 모습이어야 하는지 선언적으로 묘사.
  - React가 상태 변화에 따라 UI를 자동으로 업데이트.
  - 목적지만 말하면 운전기사가 알아서 최적 경로를 찾아가는 방식.

## 2. React에서 UI 선언하기

1. 시각적 State를 정의

- empty: 입력 필드 비어있음.
- typing: 입력 중.
- submitting: 제출 중(스피너 표시).
- success: 제출 성공(감사 메시지).
- error: 오류 메시지 표시.
-

2. State 변화를 트리거하는 요소 파악

- 휴먼 입력: 사용자가 폼을 입력하거나 제출.
- 컴퓨터 입력: 네트워크 응답 성공 또는 실패.

## 3. State를 useState로 표현

"단순함이 핵심"

필요한 시각적 State만 useState로 저장한다.

```jsx
const [answer, setAnswer] = useState("");
const [error, setError] = useState(null);
const [status, setStatus] = useState("typing"); // 'typing', 'submitting', 'success'
```

### 불필요한 State 제거

```
중복 제거 예시:
isEmpty → answer.length === 0로 대체.
isError → error !== null로 대체.
"불가능한 State" 제거:
isTyping, isSubmitting → 동시에 true 불가 -> 하나의 status 변수로 통합.
```

## 4. 이벤트 핸들러로 State 설정

사용자 입력과 같은 이벤트를 처리해 State를 변경합니다.

#### 텍스트 입력 핸들러

```javascript
function handleTextareaChange(e) {
  setAnswer(e.target.value);
}
```

사용자가 텍스트를 입력하면 상태를 업데이트

#### 폼 제출 핸들러

```javascript
async function handleSubmit(e) {
  e.preventDefault();
  setStatus("submitting");
  try {
    await submitForm(answer);
    setStatus("success");
  } catch (err) {
    setStatus("typing");
    setError(err);
  }
}
```

### 모든 상호작용을 state로 표현하게 되면 이후에 새로운 state가 추가되더라도 기존의 로직이 손상되는 것을 막을 수 있다.
