# State를 사용해 Input 다루기

## 선언형 UI와 명령형 UI
**명령형 프로그래밍**은 UI를 조작하기 위해 발생한 상황에 따라 정확한 지침을 작성해야하고 이렇게 만들어진 UI가 명령형 UI입니다.

ex)
- 폼에 무언가 입력하면 제출 버튼이 활성화 될 것이다.
- 제출 버튼을 누르면 폼과 버튼이 비활성화 되고 스피너가 나타날 것이다.
- 네트워크 요청이 성공하면 폼은 숨겨지고 "감사합니다" 라는 메시지가 나타날 것이다.
- 네트워크 요청이 실패하면 오류 메시지가 보일 것이고 폼은 다시 활성화 될 것이다.

```js
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

function handleTextareaChange() {
  if (textarea.value.length === 0) {
    disable(button);
  } else {
    enable(button);
  }
}

function hide(el) {
  el.style.display = 'none';
}

function show(el) {
  el.style.display = '';
}

function enable(el) {
  el.disabled = false;
}

function disable(el) {
  el.disabled = true;
}

function submitForm(answer) {
  // 네트워크에 접속한다고 가정해봅시다.
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (answer.toLowerCase() === 'istanbul') {
        resolve();
      } else {
        reject(new Error('Good guess but a wrong answer. Try again!'));
      }
    }, 1500);
  });
}

let form = document.getElementById('form');
let textarea = document.getElementById('textarea');
let button = document.getElementById('button');
let loadingMessage = document.getElementById('loading');
let errorMessage = document.getElementById('error');
let successMessage = document.getElementById('success');
form.onsubmit = handleFormSubmit;
textarea.oninput = handleTextareaChange;
```
위 예시는 명령형으로 프로그래밍하여 UI를 조작한 것으로 시스템이 복잡해질수록 난이도는 더욱 올라갑니다.

**선언형 프로그래밍**은 UI를 통해 보여주고 싶은 것을 선언함으로써 프로그래밍합니다. 아래 예시를 보고 명령형 프로그래밍과 비교해보세요

## 1. 컴포넌트의 다양한 시각적 state를 확인하기
- Empty: 폼은 비활성화된 제출 버튼을 가지고 있다.
- Typing: 폼은 활성화된 제출 버튼을 가지고 있다.
- Submitting: 폼은 완전히 비활성화되고 스피너가 보인다.
- Success:폼 대신 "감사합니다"라는 메시지가 보인다.
- Error: "Typing" state와 동일하지만 오류 메시지가 보인다.

## 2. 무엇이 state 변화를 트리거하는지 알아내기

state 변경을 트리거하는 두가지 유형
- 휴먼 인풋 : 버튼을 누르거나, 필드 입력, 링크 이동 등
- 컴퓨터 인풋: 네트워크 응답이 오거나, 타임아웃, 이미지 로딩 등

위에서 설명한 인풋 유형들은 state 변수를 변경하는 움직임인 것입니다. <br/>
즉 사람이 제출 버튼을 클릭하면, Submitting state를 변경해야 한다는 것이고 네트워크 응답이 성공적으로 오면 컴퓨터는 Success state를 변경하며 반대로 실패한다면 컴퓨터는 오류 메시지와 함께 Error state를 변경합니다.

> 휴먼 인풋은 이벤트 핸들러로 나타낼 수도 있습니다!

## 3. 메모리의 state를 `useState`로 표현하기

`useState`를 사용하면 state의 변화에 따라 컴포넌트를 랜더링합니다. state는 최소화하여 버그를 방지하는 것이 좋습니다!
```jsx
const [answer, setAnswer] = useState('');
const [error, setError] = useState(null);
const [isEmpty, setIsEmpty] = useState(true);
const [isTyping, setIsTyping] = useState(false);
const [isSubmitting, setIsSubmitting] = useState(false);
const [isSuccess, setIsSuccess] = useState(false);
const [isError, setIsError] = useState(false);
```
위 예제는 명령형으로 만들었던 예제를 바꾸기 위해 필요하다고 생각되는 state들을 선언한것입니다.
이는 개발 과정에서 추가 될 수도 제거할 수도 있습니다! state는 최소화 하고 꼭 필요한 state만 사용하는 것이 좋으니까요

## 4. 불필요한 state 변수 제거하기

state의 중복을 피하고 사용자에게 유효한 UI를 보여주지 않는 경우를 방지하는 과정으로 고려해봐야 할 것들이 있습니다.

- state가 역설을 일으키는가? <br/>
`isTyping`과 `isSubmitting`는 동시에 `true` 일 수 없습니다. 또한 boolean으로 만들 수 있는 state의 유효한 조합은 3개 뿐입니다. 따라서 `typing`, `submitting`, `success`을 하나의 `status`로 합칠 수 있습니다.
- 다른 state 변수에 이미 같은 정보가 있는가? <br/>
`isEmpty`와 `isTyping`은 동시에 `true`가 될 수 없습니다. 이 경우 `isEmpty`를 `answer.length === 0`으로 대체할 수 있습니다.
- 다른 변수를 뒤집었을 때 같은 정보를 얻을 수 있는가?<br/>
`isError`는 `error !== null`로 대체할 수 있습니다.

이 과정을 거친후에 남는 state는 총 7개에서 3개로 남게됩니다.
```jsx
const [answer, setAnswer] = useState('');
const [error, setError] = useState(null);
const [status, setStatus] = useState('typing'); // 'typing', 'submitting', or 'success'
//셋 중 하나를 지웠을 때 정상 작동하지 않으니 모두 필수라는 걸 알 수 있음
```

## 5. state 설정을 위한 이벤트 핸들러 연결
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
      <h2>City quiz</h2>
      <p>
        In which city is there a billboard that turns air into drinkable water?
      </p>
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
  // 네트워크에 접속한다고 가정해봅시다.
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