# 학습 내용

## 컴포넌트에 대하여

- 컴포넌트란 앱의 재사용 가능한 UI 요소
- 리액트로 구성한 웹앱은 여러 개의 컴포넌트로 구성되어 있다.
- React 컴포넌트는 JS의 함수이다.

## 컴포넌트 정의

- **컴포넌트 내보내기**

  - `export default` 접두사를 사용하여 추후 다른 파일에서 컴포넌트를 가져올 수 있게 된다.

- **함수 정의하기**

  - `function Profile() { }`을 사용하면 `Profile`이라는 이름의 JavaScript 함수를 정의할 수 있다.
  - React 컴포넌트는 함수 이름이 대문자로 시작하는 컨벤션을 지킨다.

- **JSX 마크업 추가하기**  
   다음과 같이 JSX 마크업을 추가할 수 있으며, `return` 이후에 괄호가 필요하다.

  ```jsx
  return (
    <div>
      <img src="https://i.imgur.com/MK3eW3As.jpg" alt="Katherine Johnson" />
    </div>
  );
  ```

  ## 컴포넌트 사용하기

```jsx
function Profile() {
  return <img src="https://i.imgur.com/MK3eW3As.jpg" alt="Katherine Johnson" />;
}

export default function Gallery() {
  return (
    <section>
      <h1>Amazing scientists</h1>
      <Profile />
      <Profile />
      <Profile />
    </section>
  );
}
```

## 주의사항

컴포넌트 내부에서 다른 컴포넌트를 정의해선 안된다 (매우 느리며 버그를 촉발함)
따라서 최상위 레벨에서 컴포넌트를 정의하는 것을 권장한다
