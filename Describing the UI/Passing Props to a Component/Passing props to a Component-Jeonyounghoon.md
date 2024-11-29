# props 에 대하여

비록 받는 데이터만으로 상호작용 하는것이지만, 리액트는 props를 통해 컴포넌트간 상호작용이 가능함.

## props 전달 방법

### 1. 자식 컴포넌트에 props를 전달

```jsx
export default function Profile() {
  return (
    <Avatar person={{ name: "Lin Lanying", imageId: "1bX5QH6" }} size={100} />
  );
}
```

해당 코드는 Avatar 컴포넌트에 person객체와 size를 전달함.

### 2. 자식 컴포넌트 내부에서 props 읽기

```jsx
function Avatar({ person, size }) {
  // person과 size는 이곳에서 사용가능합니다.
}
```

props는 객체라고 생각하면 편함.

가령 위의 코드에선,
`props = { person={{ name: 'Lin Lanying', imageId: '1bX5QH6' }}, size=100}`인셈
따라서 자식 컴포넌트에서 props를 받을때도 객체구조분해할당으로 받게 되는 것.

## props의 기본값 지정

값이 지정되지 않았을 때, prop에 기본값을 주길 원한다면, 변수 바로 뒤에 = 과 함께 기본값을 넣어 구조 분해 할당을 해줄 수 있음.

```jsx
function Avatar({ person, size = 100 }) {
  // ...
}
```

## 자식을 JSX로 전달 할 수도있음.

```jsx
<Card>
  <Avatar />
</Card>
```
