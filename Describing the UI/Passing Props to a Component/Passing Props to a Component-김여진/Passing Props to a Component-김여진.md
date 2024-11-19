### 컴포넌트에 props 전달하기

- React 컴포넌트는 부모컴포넌트에서 자식컴포넌트로 props를 전달하며 통신한다.

- React 컴포넌트 함수는 props 객체(JavaScript객체) 하나를 매개변수로 받음

- Props는 단순히 읽기 전용 스냅샷으로, 렌더링 할 때마다 새로운 버전의 props를 받음

- React의 단방향 데이터 흐름을 따르고, Virtual DOM 최적화와 코드 안정성을 높이기 위해 Props는 변경할 수 없음! 상호작용이 필요한 경우 state 사용하기

```jsx
// 자식 컴포넌트, 래퍼(컨테이너) 컴포넌트
// props객체를 전달받아 props.children을 렌더링하는 역할
function Card({ children }) {
  return (
    <div className="card">
      {children}
    </div>
  );
}
//+React는 내부적으로 아래 객체를 생성해 전달함
{
  children: [
    <Avatar
        person={{ name: 'Lin Lanying', imageId: 'S3Gh0Zp' }}
        size={120}
    />
  ]
}

//부모 컴포넌트
export default function Profile() {
  return (
    <Card>
      <Avatar
        person={{ name: 'Lin Lanying', imageId: 'S3Gh0Zp' }}
        size={120}
      />
    </Card>
  );
}

//+React는 내부적으로 아래 객체를 생성해 전달함
{
  person: { name: 'Lin Lanying', imageId: 'S3Gh0Zp' },
  size: 120
}

//자식 컴포넌트
//특정 prop이 전달되지 않았을때(undefined일때) 사용할 default 값 선언 가능 
function Avatar({ perdon, size=100 }) {
  return (
    <img
      className="avatar"
      alt={person.name}
      src={`https://i.imgur.com/${person.imageId}s.jpg`}
      width={size}
      height={size}
    />
  );
}

//아래와 동일함
function Avatar(props) {
  let person = props.person;
  let size = props.size===undefined ?  100 : props.size;
  // ...
}
```