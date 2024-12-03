### 선연형 UI vs 명령형 UI

`명령형 UI`를 구현할땐 발생한 상황에 대한 세밀한 지침을 순서대로 알려야 함

e.g. 폼을 명령형 UI로 구현하는 경우

폼에 무언가를 입력하면 “제출” 버튼이 활성화됨
”제출” 버튼을 누르면 폼과 버튼이 비활성화되고 스피너가 나타남
네트워크 요청이 성공하면 폼은 숨겨질 것이고 “감사합니다.” 메시지가 나타남
네트워크 요청이 실패하면 오류 메시지가 보일 것이고 폼은 다시 활성화됨

```js
// DOM API 사용
async function handleFormSubmit(e) {
  e.preventDefault();
  disable(textarea);
  disable(button);
  show(loadingMessage);
  hide(errorMessage);
  try {
    await submitForm(textarea.value);
    show(successMessage);
    hide(form);
  } catch (err) {
    show(errorMessage);
    errorMessage.textContent = err.message;
  } finally {  
    hide(loadingMessage);
    enable(textarea);
    enable(button);
  }
}
...
```

`선언형 UI`를 구현할 땐 UI를 직접 조작하는 게 아니라 시각적 state로 `선언`하기만 하면 됨. 그러면 React가 UI를 자동으로 업데이트해줌 

새로운 시각적 state가 추가되도 기존 로직 손상x, 상호작용 로직을 변경하지 않고도 각각의 state에 표시되는 항목 변경 가능

1. 컴포넌트의 모든 시각적 state 확인

```
Empty: 폼은 비활성화된 “제출” 버튼을 가짐
Typing: 폼은 활성화된 “제출” 버튼을 가짐
Submitting: 폼은 완전히 비활성화되고 스피너가 보임
Success: 폼 대신에 “감사합니다” 메시지가 보임
Error: “Typing” state와 동일하지만 오류 메시지가 보임
```

2. 휴먼이나 컴퓨터가 어떻게 state 변화를 트리거하는지 생각

```
(휴먼 인풋)
텍스트 인풋을 변경하면 텍스트 상자가 비어있는지 여부에 따라 state를 Empty에서 Typing 또는 그 반대로 변경해야 함
제출 버튼을 클릭하면 Submitting state를 변경해야 함

(컴퓨터 인풋)
네트워크 응답이 성공적으로 오면 Success state를 변경해야 함
네트워크 요청이 실패하면 해당하는 오류 메시지와 함께 Error state를 변경해야 힘
```

3. `useState`로 state를 모델링

반드시 필요한 state만 선언하는 게 핵심

```js
const [answer, setAnswer] = useState('');
const [error, setError] = useState(null);
const [isEmpty, setIsEmpty] = useState(true);
const [isTyping, setIsTyping] = useState(false);
const [isSubmitting, setIsSubmitting] = useState(false);
const [isSuccess, setIsSuccess] = useState(false);
const [isError, setIsError] = useState(false);
```

4. state 리팩토링

state로 인해 사용자에게 유효하지 않은 UI를 보여주는 것을 방지하는 단계

- state가 역설을 일으키는가?
isTyping,isSubmitting, isSuccess가 동시에 true일 수는 없으니 하나의 status로 합침
isEmpty, isTyping은 동시에 true가 될 수 없음.
- 다른 변수로 대체할 수 있는가?
isEmpty는 answer.length===0으로 대체 가능.
isError는 error !== null로 대체 가능.

```js
const [answer, setAnswer] = useState('');
const [error, setError] = useState(null);
const [status, setStatus] = useState('typing'); // 'typing', 'submitting', or 'success'
// ✅ 어느 하나를 지웠을 때 정상적으로 작동하지 않으니 이 state들은 필수암
```

5. state 설정을 위해 이벤트 핸들러 연결 

```jsx
import { useState } from 'react';

export default function Form() {
  const [answer, setAnswer] = useState('');
  const [error, setError] = useState(null);
  const [status, setStatus] = useState('typing');

  if (status === 'success') {
    return <h1>That's right!</h1>
  }

  async function handleSubmit(e) {
    e.preventDefault();
    setStatus('submitting');
    try {
      await submitForm(answer);
      setStatus('success');
    } catch (err) {
      setStatus('typing');
      setError(err);
    }
  }

  function handleTextareaChange(e) {
    setAnswer(e.target.value);
  }

  return (
    <>
      <form onSubmit={handleSubmit}>
        <textarea
          value={answer}
          onChange={handleTextareaChange}
          disabled={status === 'submitting'}
        />
        <br />
        <button disabled={
          answer.length === 0 ||
          status === 'submitting'
        }>
          Submit
        </button>
        {error !== null &&
          <p className="Error">
            {error.message}
          </p>
        }
      </form>
    </>
  );
}

function submitForm(answer) {
  // 네트워크에 접속한다고 가정
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      let shouldError = answer.toLowerCase() !== 'lima'
      if (shouldError) {
        reject(new Error('Good guess but a wrong answer. Try again!'));
      } else {
        resolve();
      }
    }, 1500);
  });
}

```

