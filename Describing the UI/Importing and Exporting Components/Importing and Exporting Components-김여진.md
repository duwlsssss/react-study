### 컴포넌트 import 및 export 하기

- 컴포넌트를 추가할 js 파일을 만들고
- 새로 만든 파일에서 함수 컴포넌트를 export함
- 컴포넌트를 사용할 파일에서 import함

```jsx
// Sidebar.jsx
const SideBar = () => {
  return (
    <S.SidebarContainer>
      <S.SidebarLink to='/search'>
        <S.SidebarLinkInner>
          <IoSearch size={25} /> 찾기
        </S.SidebarLinkInner>
      </S.SidebarLink>
      <S.SidebarLink to='/categories'>
        <S.SidebarLinkInner>
          <PiFilmSlate size={25} /> 영화
        </S.SidebarLinkInner>
      </S.SidebarLink>
    </S.SidebarContainer>
  );
};
export default Sidebar;

// Layout.jsx
import React from "react";
import { Outlet } from "react-router-dom";
import { Navbar, SideBar } from "../../components";
import * as S from './RootLayout.styles';

const RootLayout = () => {
  return (
      <>
        <Navbar/>
        <S.ContentContainer>
          <SideBar />
          <S.OutletContainer>
            <Outlet />
          </S.OutletContainer>
        </S.ContentContainer>
    </>
  )
}

export default RootLayout;
//
```

### + default vs named export(import)
- 한 파일에서 하나의 default export만 존재할 수 있고 / named export는 여러 개가 존재할 수 있음
- Default import 구문: `import [컴포넌트명] from '[경로]'` / Named import 구문: `import {[컴포넌트명]} from '[경로]'`
- Default import에서 컴포넌트명은 실제 컴포넌트 이름이 아니어도 같은 default export 값을 가져옴(그래도 컴포넌트 이름을 유의미하게 짓는게 중요) / 반대로 Named import는 양쪽 파일에서 사용하려는 이름이 같아야 하기 때문에 named import라고 불림

> 일반적으로 한 파일에서 하나의 컴포넌트만 export 할 때 default export 방식을 사용하고, 여러 컴포넌트를 export 할 경우엔 named export 방식을 사용
