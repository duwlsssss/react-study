### 리스트 렌더링

배열 데이터를 사용해 유사한 컴포넌트를 생성할 수 있음

```jsx
import { recipes } from './data.js';

function Recipe({name,ingredients}){
  return(
    <>
      <h2>{name}</h2>
      <ul>
        {ingredients.map(ingredient =>
          <li key={ingredient}>
            {ingredient}
          </li>
        )}
      </ul>
    </>
  );
}
export default function RecipeList() {
  return (
    <div>
      <h1>Recipes</h1>
      {recipes.map(recipe => 
        <Recipe 
          key={recipe.id} {/* ⚠ map() 내부 jsx 엘리먼트에는 고유한 key 속성을 지정해야 함*/}
          {...recipe}
        />
      )}
    </div>
  );
}
```

#### ✅ Key 
- Key는 React가 각 요소를 고유하게 식별할 수 있도록 돕는 역할
즉 Key는 Virtual DOM의 Diffing 과정에서 요소들의 변경 여부를 빠르게 파악하기 위해 사용
- Key값은 형제간에 고유해야함 but 다른 배열에선 같은 키 사용 가능
- 배열의 인덱스값같이 변경될 수 있는 값이면 안됨
- 렌더링 중에 key 생성 금지(렌더링 중 생성된 Key는 매번 새 값으로 평가될 수 있어 비효율적)
- key는 컴포넌트에 prop으로 사용x, key는 React에서 힌트처럼 작동하는 거임