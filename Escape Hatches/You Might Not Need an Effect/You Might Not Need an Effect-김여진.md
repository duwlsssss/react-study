### 불필요한 Effect가 제거하기

- Effect가 필요하지 않은 경우
  - 렌더링을 위해 데이터를 변환하는 경우
  ```js
  // e.g. items prop이 다른 배열을 받을 때마다 selection을 null로 재설정하려고 함
  function List({ items }) {
    const [isReverse, setIsReverse] = useState(false);
    // 🔴 bad
    // state를 업데이트할 때 React는 먼저 컴포넌트 함수를 호출해 화면에 표시될 내용을 계산 -> 변경사항을 DOM에 commit하며 화면 업데이트 -> Effect 실행, Effect도 state를 업데이트 하면 이 과정이 다시 실행됨 
    // 기존 props나 state에서 계산할 수 있으면, 그것을 state에 넣지 말고 컴포넌트의 최상위 레벨에서 렌더링 중에 계산하게 하기
     const [selection, setSelection] = useState(null);
     useEffect(() => {
      setSelection(null);
    }, [items]);
    // ...
    // 🟢 Good
    const [selectedId, setSelecedId] = useState(null);
    const selection = items.find(item => item.id === selectedId) ?? null;
  }
  ```
  - 사용자 이벤트를 처리하는 경우
  ```js
  // e.g. 사용자가 폼을 작성하고 Submit 버튼을 클릭하면 POST 요청을 보냄
  function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');

    // 🟢 Good: 컴포넌트가 사용자에게 표시되었기 때문에 실행되어야 하는 코드에만 Effect를 사용
    useEffect(() => {
      post('/analytics/event', { eventName: 'visit_form' });
    }, []);

    // 🔴 Bad: Effect에 사용자 이벤트 처리 코드를 넣음
    const [jsonToSubmit, setJsonToSubmit] = useState(null);
    useEffect(() => {
      if (jsonToSubmit !== null) {
        post('/api/register', jsonToSubmit);
      }
    }, [jsonToSubmit]);

    function handleSubmit(e) {
      e.preventDefault();
      setJsonToSubmit({ firstName, lastName });
    }
    // ...

    // 🟢 Good: 사용자 이벤트별 로직은 이벤트 핸들러에 씀
    function handleSubmit(e) {
      e.preventDefault();
      post('/api/register', jsonToSubmit);
    }
    // ...
  }
  ```
+ Effect를 사용하는 코드를 사용자 정의 Hook으로 추출하면 
컴포넌트에서 원시 useEffect 호출을 줄일 수 있음 -> 유지보수 👍
```js
// e.g. 데이터 가져오는 로직을 useData 훅으로 분리
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
// 대신
function SearchResults({ query }) {
  const [page, setPage] = useState(1);
  const params = new URLSearchParams({ query, page });
  const results = useData(`/api/search?${params}`);

  function handleNextPageClick() {
    setPage(page + 1);
  }
  // ...
}

function useData(url) {
  const [data, setData] = useState(null);
  useEffect(() => {
    let ignore = false;
    fetch(url)
      .then(response => response.json())
      .then(json => {
        if (!ignore) {
          setData(json);
        }
      });
    return () => {
      ignore = true;
    };
  }, [url]);
  return data;
}
```