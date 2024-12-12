### 데이터 가져오기
 

많은 앱이 데이터 가져오기를 시작하기 위해 Effect를 사용한다.

아래와 같은 데이터를 가져오는 Effect를 작성하는 것은 매우 일반적이다.

 
```
function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const [page, setPage] = useState(1);

  useEffect(() => {
    // 🔴 피하세요: 정리 로직 없이 가져오기
    fetchResults(query, page).then(json => {
      setResults(json);
    });
  }, [query, page]);

  function handleNextPageClick() {
    setPage(page + 1);
  }
  // ...
}
```
데이터 가져오기를 이벤트 핸들러로 옮길 필요는 없다.

하지만. hello를 입력한다고 했을 때 h he hel 이 아닌 h hel he로 응답을 받을 수도 있는 것이다.

이것을 "경쟁 조건"이라 하는데, 서로 다른 두 요청이 서로 "경쟁"하여 예상과 다른 순서로 도착하는 것을 말한다.

 

경쟁 조건을 수정하려면 오래된 응답을 무시하는 정리 함수를 추가해야한다.

 ```

function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const [page, setPage] = useState(1);
  useEffect(() => {
    let ignore = false;
    fetchResults(query, page).then(json => {
      if (!ignore) {
        setResults(json);
      }
    });
    return () => {
      ignore = true;
    };
  }, [query, page]);

  function handleNextPageClick() {
    setPage(page + 1);
  }
  // ...
}
 ```

이렇게 작성하면 Effect가 데이터를 가져올 때 마지막으로 요청된 응답을 제외한 모든 응답이 무시된다.

 

### 데이터 가져오기를 구현할 때  다음의 조건들도 고려해봐야한다.

- 경쟁 조건 처리
- 응답 캐싱(사용자가 뒤로가기 버튼을 클릭하여 이전 화면을 즉시 볼 수 있도록)
- 서버에서 데이터를 가져오는 방법(초기 서버 렌더링 HTML에 스피너 대신 가져온 콘텐츠가 포함되도록)
- 네트워크 워터풀을 피하는 방법(자식이 모든 부모를 기다리지 않고 데이터를 가져올 수 있도록)

이러한 문제는 React뿐만 아니라 모든UI라이브러리에 적용된다

따라서 모던 프레임워크는 Effect에서 데이터를 가져오는 것보다 더 효율적인 내장 데이터 가져오기 매커니즘을 제공한다.

`const results = useData(`/api/search?${params}`);`


전체 컴포넌트 트리의 state를 초기화하려면 다른 key를 전달
컴포넌트가 표시되어 실행되는 코드는 Effect에 있어야하고 나머지는 이벤트에 있어야 한다.