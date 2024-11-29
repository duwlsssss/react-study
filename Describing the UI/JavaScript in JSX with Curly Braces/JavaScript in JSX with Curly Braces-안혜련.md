# JavaScript in JSX with Curly Braces
JSX를 사용하면 JavaScript 파일에 HTML과 비슷한 마크업을 작성하여 렌더링 로직과 콘텐츠를 같은 곳에 놓을 수 있다.

 때로는 JavaScript 로직을 추가하거나 해당 마크업 내부의 동적인 프로퍼티를 참조하고 싶을 수 있다. 
 
 이 상황에서는 JSX에서 중괄호를 사용하여 JavaScript를 사용할 수 있습니다.

## 따옴표로 문자열 전달하기

문자열 어트리뷰트를 JSX에 전달하려면 작은따옴표나 큰따옴표로 묶어야 한다.

```jsx
export default function Avatar() {
  return (
    <img
      className="avatar"
      src="https://i.imgur.com/7vQD0fPs.jpg"
      alt="Gregorio Y. Zara"
    />
  );
}
```

src나 alt를 동적으로 지정하려면 따옴표를 {}로 대체하면 된다.

```jsx
export default function Avatar() {
  const avatar = 'https://i.imgur.com/7vQD0fPs.jpg';
  const description = 'Gregorio Y. Zara';
  return (
    <img
      className="avatar"
      src={avatar}
      alt={description}
    />
  );
}
```

JSX 내부에서는 중괄호로 두 가지 방법만 사용할 수 있다.

1. JSX 태그 안에 직접 텍스트로 사용하기(⭕️) 

`<{tag}>Gregorio Y. Zara's To Do List</{tag}> (❌) `

2. {name}'s To Do List
3. = 기호 바로 뒤에 오는 속성src={avatar}는 변수 아바타를 읽지만, src="{avatar}"는 문자열 "{avatar}"를 전달한다.

1. `=` 바로 뒤에 오는 **어트리뷰트**: `src={avatar}`는 `avatar` 변수를 읽지만 `src="{avatar}"`는 `"{avatar}"` 문자열을 전달한다.

## 이중 중괄호 사용

문자열, 숫자, JS 표현식 외에도 JSX로 객체를 전달할 수 있다.

객체는 중괄호로 표시하기 때문에 JSX에서 JS 객체를 전달하려면 다른 중괄호 쌍으로 객체를 감싸야 한다.

```jsx
export default function TodoList() {
  return (
    <ul style={{
      backgroundColor: 'black',
      color: 'pink'
    }}>
      <li>Improve the videophone</li>
      <li>Prepare aeronautics lectures</li>
      <li>Work on the alcohol-fuelled engine</li>
    </ul>
  );
}

```

위와 같이 주로 인라인 CSS 스타일에서 볼 수 있다.

React에서는 유지보수 및 가독성을 위해 인라인 스타일을 지양하지만 필요한 경우 style 어트리뷰트에 객체를 전달한다.

이런 경우 인라인 style 프로퍼티는 카멜 케이스로 작성된다.

> JSX에서 `{{` 와 `}}` 를 본다면 JSX 중괄호 안의 객체에 불과하다는 것! 

---
요약 정리 

- 따옴표 안의 JSX 속성은 문자열로 전달된다.
- 중괄호를 사용하면 JS 로직과 변수를 마크업으로 가져올 수 있다.
- 중괄호는 JSX 태그 콘텐츠 내부 또는 속성의 = 뒤에서 작동한다.
- {{}}는 JSX 중괄호 안에 들어 있는 JS 객체다.