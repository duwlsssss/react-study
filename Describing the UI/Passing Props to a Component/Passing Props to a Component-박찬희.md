# Passing Props to a Component
React 컴포넌트는 props를 이용해 서로 통신합니다. 모든 부모 컴포넌트는 props를 줌으로써 몇몇의 정보를 자식 컴포넌트에게 전달할 수 있습니다. props는 HTML 어트리뷰트를 생각나게 할 수도 있지만, 객체, 배열, 함수를 포함한 모든 JavaScript 값을 전달할 수 있습니다.

## Familiar props
props는 JSX태그를 전달하는 정보입니다. 예를 들어, `className`, `src`, `width`, `height`는 `<img>`태그에 전달할 수 있습니다.
```jsx
function Avatar() {
  return (
    <img
      className="avatar"
      src="https://i.imgur.com/1bX5QH6.jpg"
      alt="Lin Lanying"
      width={100}
      height={100}
    />
  );
}

export default function Profile() {
  return (
    <Avatar />
  );
}
```
`<img>`태그에 전달할 수 있는 props는 미리 정의되어 있습니다. (ReactDOM은 HTML표준을 준수합니다) 자신이 생성한 `<Avatar>`와 같은 어떤 컴포넌트든 props를 전달 할 수 있습니다.

## 컴포넌트에 props 전달하기
위 예시 코드를 보면 `Profile` 컴포넌트는 자식 컴포넌트인 Avatar에 어떠한 props도 전달하지 않았습니다. 앞으로 두 단계에 걸쳐 `Avatar`에 props를 전달할 수 있습니다.

### 1단계: 자식 컴포넌트에 props 전달하기
Avatar 컴포넌트에 props를 전달합니다.
```jsx
export default function Profile() {
  return (
    <Avatar
      person={{ name: 'Lin Lanying', imageId: '1bX5QH6' }}
      size={100}
    />
  );
}
```

이제 `Avatar` 컴포넌트 안에서 props를 읽을 수 있습니다.

### 2단계: 자식 컴포넌트 내부에서 props 읽기

이러한 props는 `function Avatar` 바로 뒤에 있는 `({})` 안에 그들의 이름인 `person, size` 등을 쉼표로 구분함으로써 읽을 수 있습니다. 그러면 `Avatar` 코드 내에서 변수처럼 사용할 수 있습니다.
```jsx
function Avatar({ person, size }) {
  // person과 size는 이곳에서 사용가능합니다.
}
```
`Avatar`에 랜더링을 위해 `person` 과 `size` props를 사용하는 로직을 추가하면 완료됩니다. 

아래 예제는 props를 이용하기 위해 `getImageUrl`함수를 다른 파일에서 export해왔습니다.
```jsx
// App.js
import { getImageUrl } from './utils.js';

function Avatar({ person, size }) {
  return (
    <img
      className="avatar"
      src={getImageUrl(person)}
      alt={person.name}
      width={size}
      height={size}
    />
  );
}

export default function Profile() {
  return (
    <div>
      <Avatar
        size={50}
        person={{ 
          name: 'Lin Lanying',
          imageId: '1bX5QH6'
        }}
      />
    </div>
  );
}

```
```jsx
// utils.js
export function getImageUrl(person, size = 's') {
  return (
    'https://i.imgur.com/' +
    person.imageId +
    size +
    '.jpg'
  );
}
```

props를 사용하면 부모 컴포넌트와 자식 컴포넌트를 독립적으로 생각할 수 있습니다. `Avatar`가 props를 어떻게 사용하는지 생각할 필요 없이 `Profile`의 `person` 또는 `size` props를 수정할 수 있습니다. 마찬가지로 `Profile`을 보지 않고도 `Avatar`가 props를 사용하는 방식을 바꿀 수 있습니다.

props는 조절할 수 있는 손잡이라고 생각하면 됩니다. props는 함수의 인수와 동일한 역할을 합니다. 사실 props는 컴포넌트에 대한 유일한 인자입니다. React 컴포넌트 함수는 하나의 인자, 즉 props 객체를 받습니다.

## prop의 기본값 지정하기
값이 지정되지 않았을 때, prop에 기본값을 주길 원한다면, 변수 뒤에 `=`과 함께 기본값을 넣어 구조 분해 할당을 해줄 수 있습니다.
```jsx
function Avatar({ person, size = 100 }) {
  // ... 
  // 따로 size prop이 없이 랜더링 된다면 size는 100으로 설정됩니다.
}
```
이 기본값은 size prop이 없거나 `size={undefined}`로 전달될 때 사용됩니다. 그러나 `size={null}` 또는 `size={0}`으로 전달된다면, 기본값은 사용되지 않습니다.

## JSX spread 문법으로 props 전달하기
반복적인 코드는 가독성을 높일 수 있다는 점에서 잘못된 것은 아닙니다. 하지만 때로는 간결함이 중요할 때도 있습니다. 일부 컴포넌트는 그들의 모든 props를 자식 컴포넌트에 전달합니다. props를 직접 사용하지 않기 때문에 보다 간결한 "spread"문법을 사용하는 것이 합리적일 수 있습니다.
```jsx
function Profile(props) {
  return (
    <div className="card">
      <Avatar {...props} />
    </div>
  );
}
```
이렇게 하면 `Profile`의 모든 props를 각각의 이름을 나열하지 않고 `Avatar`로 전달합니다.

 다른 모든 컴포넌트에 이 구문을 사용한다면 문제가 있기 때문에 **spread 문법은 제한적으로 사용해야합니다**. 이는 종종 컴포넌트들을 분할하여 자식을 JSX로 전달해야 함을 나타냅니다.

 ## 자식을 JSX로 전달하기
 JSX 태그 내에 콘텐츠를 중첩하면, 부모 컴포넌트는 해당 콘텐츠를 `children`이라는 prop으로 받을 것입니다. 예를 들어, 아래의 Card 컴포넌트는 `<Avatar />`로 설정된 `children` prop을 받아 이를 래퍼 div에 렌더링 할 것입니다.
 ```jsx
//  Avatar.js 파일은 앞서 나온 예제의 Avatar 컴포넌트와 동일합니다
 import Avatar from './Avatar.js';

function Card({ children }) {
  return (
    <div className="card">
      {children}
    </div>
  );
}

export default function Profile() {
  return (
    <Card>
      <Avatar
        size={100}
        person={{ 
          name: 'Katsuko Saruhashi',
          imageId: 'YfeOqp2'
        }}
      />
    </Card>
  );
}
 ```
 `<Card>`내부의 `<Avatar>`를 텍스트로 바꾸어 `<Card>` 컴포넌트가 중첩된 콘텐츠를 어떻게 감싸는지 확인해보세요. 그 내부에 무엇이 랜더링 되는지 `<Card>`컴포넌트에서는 알 필요는 없습니다.
 `children` prop을 가지고 있는 컴포넌트는 부모 컴포넌트가 임의의 JSX로 “채울” 수 있는 “구멍”이 있는 것으로 생각할 수 있습니다. 패널, 그리드 등의 시각적 래퍼에 종종 `children` prop를 사용합니다.

 ## 시간에 따라 props가 변하는 방식
아래의 `Clock` 컴포넌트는 부모 컴포넌트로부터 `color`와 `time`이라는 두 가지 props를 받습니다. (부모 컴포넌트의 코드는 아직 자세히 다루지 않을 state를 사용하기 때문에 생략합니다.)
```js
export default function Clock({ color, time }) {
  return (
    <h1 style={{ color: color }}>
      {time}
    </h1>
  );
}
```
이 예시는 생략된 코드가 많아 정확히 알긴 어렵지만 컴포넌트가 시간에 따라 다른 props를 받을 수 있음을 보여줍니다. Props는 항상 고정되어 있지 않습니다. 여기서 `time` prop은 매초 변경되고, `color` prop은 다른 색상을 선택하면 변경됩니다. Props는 컴포넌트의 데이터를 처음에만 반영하는 것이 아니라 모든 시점에 반영합니다.

그러나 props는 컴퓨터 과학에서 “변경할 수 없다”라는 의미의 불변성을 가지고 있습니다. 컴포넌트가 props를 변경해야 하는 경우(예: 사용자의 상호작용이나 새로운 데이터에 대한 응답), 부모 컴포넌트에 다른 props, 즉 새로운 객체를 전달하도록 “요청”해야 합니다. 그러면 이전의 props는 버려지고, 결국 자바스크립트 엔진은 기존 props가 차지했던 메모리를 회수하게 됩니다.

”props 변경”을 시도하지 마세요. 선택한 색을 변경하는 등 사용자 입력에 반응해야 하는 경우에는 “set state”가 필요할 것입니다.