### JSX: JavaScript에 마크업 넣기 

- JSX는 JavaScript를 확장한 문법으로, 일반 JavaScript 객체를 반환함

- 사용 이유: 옛날에는 HTML,CSS,JS로 각각의 역할과 파일을 명확히 구분했으나 앱이 복잡해지며 React에사는 컴포넌트 안에 서로 관련있는 렌더링 로직과 마크업을 그룹화함 -> 변화가 생길때 렌더링 로직, 마크업이 동기화 유지됨, 반대로 관련없는 항목들은 분리해 관리 가능

- React 컴포넌트는 JSX라는 확장된 문법을 사용하여 마크업을 나타냄

```jsx
export const Category = () => {

  ...

// ✅ 컴포넌트에서 여러 요소를 반환하려면 하나의 부모 태그로 감싸야 함(Fragment: HTML 트리에 흔적을 남기지 않고 그룹화해줌) 
// 하나의 함수니까 하나의 객체를 반한하는 것임
  return (
   <>
      <S.Title>카테고리</S.Title>
      <S.CardList>
        {categories.map((category) => (
          <S.Card 
            key={category.id} 
            onClick={() => navigate(`/categories/${category.category}`, {
              replace: false,
            })}
          >
            <S.CardImg 
              src={category.image} 
              alt={category.label}
            />
            <S.CardLabel>{category.label}</S.CardLabel>
          </S.Card>
        ))}
      </S.CardList>
   </>
  )
}
```

<br/>

➕ JSX는 JS로 바뀌고 JSX에서 작성된 어트리뷰트는 JS객체의 키가 돼 변수처럼 쓰임 / JS 변수엔 명명 제약이 있음 - 예약어 사용불가, `-`포함 불가
-> 어트리뷰트를 캐멀 케이스로 작성(stroke-width 대신 strokeWidth, class 대신 className)(aria-*와 data-*형식의 어트리뷰트 제외)

이러한 모든 어트리뷰트는 React DOM 엘리먼트에서 찾을 수 있음

