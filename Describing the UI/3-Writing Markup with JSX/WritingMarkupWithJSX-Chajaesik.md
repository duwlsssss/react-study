# JSX 정의
JSX는 JS를 확장한 문법으로, JS 파일을 HTML과 비슷하게 마크업 작성할 수 있게 한다.

## 학습 내용
- React에서 마크업과 렌더링 로직을 같이 사용하는 이유
- JSX의 규칙

### React에서 마크업과 렌더링 로직을 같이 사용하는 이유
웹은 HTML로 내용을, CSS로 디자인을, JS로 로직을 기반으로 만들어져왔다.  
그러나 웹이 인터랙티브(상호 활동적)해지면서 로직이 내용을 결정하게 되었다.  
이것이 컴포넌트 안에 렌더링 로직과 마크업이 같이 사용되는 이유이다.

### JSX의 규칙

1. **하나의 루트 엘리먼트로 반환**
   ```jsx
   <>
     {/* Fragment, 브라우저상의 HTML 트리 구조에서 흔적을 남기지 않고 그룹화 */}
     <h1>Hedy Lamarr's Todos</h1>
     <img
       src="https://i.imgur.com/yXOvdOSs.jpg"
       alt="Hedy Lamarr"
       className="photo"
     />
     <ul>
       ...
     </ul>
   </>
   ```

2. **모든 태그는 닫아주기**
   JSX에서는 태그를 명시적으로 닫아야 한다.
   ```jsx
   <>
     <img /> {/* 자체적으로 닫아주는 태그 작성 방법 */}
     <ul>
       <li></li> {/* 래핑 태그 작성 방법 */}
     </ul>
   </>
   ```

3. **대부분 캐멀 케이스로 명명**
   ```javascript
   // stroke-width 대신
   strokeWidth // 사용
   ```
