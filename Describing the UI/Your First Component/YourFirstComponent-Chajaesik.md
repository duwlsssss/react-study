# 학습 내용

## 컴포넌트란?
- **UI 구성요소:** HTML, CSS, JS로 동작하는 모든 것이 UI의 기반이 된다.
- React를 사용하면 앱의 재사용 가능한 UI 요소인 사용자 정의     **컴포넌트**로 결합할 수 있다.

## React에서 컴포넌트의 역할

### 예시 코드
```
<PageLayout>  
  <NavigationHeader>
    <SearchBar />
    <Link to="/docs">Docs</Link>
  </NavigationHeader>
  <Sidebar />
  <PageContent>
    <TableOfContents />
    <DocumentationText />
  </PageContent>
</PageLayout>
```

## 컴포넌트 정의하기
```
export default function Profile() {
  return (
    <img src="https://i.imgur.com/MK3eW3Am.jpg" alt="Katherine Johnson" />
  );
}
```

## 컴포넌트를 빌드하는 방법
1. **컴포넌트 내보내기 (export)**
2. **함수 정의하기 (function () {})**
3. **마크업 추가하기**

```
return <img src="https://i.imgur.com/MK3eW3As.jpg" alt="Katherine Johnson" />;

// return 키워드와 한 줄에 있지 않다면...

return (
  <div>
    <img src="https://i.imgur.com/MK3eW3As.jpg" alt="Katherine Johnson" />
  </div>
);
```

## 컴포넌트 사용 예시

```
function Profile() {
  return (
    <img src="https://i.imgur.com/MK3eW3As.jpg" alt="Katherine Johnson" />
  );
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
## 주의사항 !!
 컴포넌트는 다른 컴포넌트를 렌더링할 수 있지만, 정의를 중첩해서는 안 된다.

 ```
export default function Gallery() {
  // 🔴 절대 컴포넌트 안에 다른 컴포넌트를 정의하면 안 됩니다!
  function Profile() {
    // ...
  }
  // ...
}

export default function Gallery() {
  // ...
}

// ✅ 최상위 레벨에서 컴포넌트를 선언합니다
function Profile() {
  // ...
}
```