### useEffect

Effect는 렌더링 자체에 의해 발생하는 부수 효과를 뜻함 e.g.채팅 컴포넌트에서 메시지를 보내는 특정 상호작용은 이벤트, 서버 연결 설정은 Effect

Effect는 주로 컴포넌트를 React를 벗어난 외부 시스템과 동기화하기 위해 사용 - 브라우저 API, 서브 파티 위젯, 네트워크 등 

Effect는 반응형 코드 블록임. 내부에서 읽은 값이 변경되면 다시 동기화함(필요할때마다 동기화 실행) <-> 이벤트는 상호작용당 한번만 실행됨

`useEffect` 훅으로 컴포넌트 내에 Effect를 선언함
- 기본적으로 모든 Effect는 commit 이후에 실행됨_모든 렌더링 이후 실행 
- 구성요소
  - 의존성 배열: useEffect 훅의 두번째 인자로 Effect를 다시 실행할 조건이 되는 변수(이전 렌더링의 것과 바뀌었을때만 Effect를 다시 실행)를 담은 배열을 넣어 필요할때만 실행할 수 있음
  Props, state, 이들로 부터 렌더링 중에 계산되는 컴포넌트 본문의 변수는 리렌더링으로 인해 바뀔 수 있으므로 반응형 -> 의존성 배열에 넣기

  반면 전역 변수, ref 등 변경할 수 있는 값은 종속성이 될 수 없음. 의존성 배열에 넣으면 변경할 수 있는 데이터를 읽는 게 돼 렌더링의 순수성을 깨뜨림
  ```js
  useEffect(() => {
  // 모든 렌더링 후에 실행
  });

  useEffect(() => {
    // 컴포넌트가 마운트될 때만 실행
  }, []);

  useEffect(() => {
  // 컴포넌트가 마운트될 때, 렌더링 이후에 a 또는 b 중 하나라도 변경된 경우에 실행
  }, [a, b]);
  ```
  - cleanup function: return문으로 함수를 반환해 Effect가 수행하던 작업을 중단하거나 되돌림, strict mode에서 컴포넌트가 두 번 렌더링 되니까 이거 확인해 클린업 작업이 필요한지 판단 - effect를 한 번 더 시작하고 중지해 정리를 잘 구현했는지 확인을 도와줌
  ```js
  import { useState, useEffect } from 'react';
  import { createConnection } from './chat.js'; // connect(), disconnect()를 반환하는 api

  export default function ChatRoom({ roomId }) {
    useEffect(() => {
      const connection = createConnection();
      connection.connect();
      return () => connection.disconnect(); // roomId 가 변경되면 클린업 함수로 이전 동기화 중지 후 다시 동기화 
      // 
    }, [roomId]);
    return <h1> {roomId}님 채팅에 오신걸 환영합니다!</h1>;
  }

  // ✅ Effect가 이벤트를 구독하면 클린업 함수에서 구독을 해지해야함
  useEffect(() => {
    function handleScroll(e) {
      console.log(window.scrollX, window.scrollY);
    }
    window.addEventListener('scroll', handleScroll);
    return () => window.removeEventListener('scroll', handleScroll); // 한 번에 하나의 활성 구독만 존재하게 만듦
  }, []);

  // ✅ Effect가 애니메이션을 트리거하는 경우 클린업 함수에서 초기값으로 재설정해주기
  useEffect(() => {
    const node = ref.current;
    node.style.opacity = 1; // 애니메이션 트리거
    return () => {
      node.style.opacity = 0; // 초기화
    };
  }, []);

  // ✅ Effect가 데이터 fetching을 하면 클린업 함수에서 fetch를 중단 or 결과를 무시해야함
  useEffect(() => {
    let ignore = false;

    async function startFetching() {
      const json = await fetchTodos(userId);
      if (!ignore) {
        setTodos(json);
      }
    }

    startFetching();

    return () => { ignore = true; }; 
    // 'Bob'에 대해 fetchTodos를 실행
    // userId가 'Taylor'로 바뀌면 이전(Bob)의 Effect가 정리되며 ignore = true가 됨
    // 'Taylor'의 fetchTodos가 Bob의 fetchTodos보다 먼저 완료됨
    // 'Taylor' 렌더링의 Effect가 setTodos 호출
    // 'Bob' 렌더링의 Effect는 ignore 플래그가 true로 설정됐기 때문에 아무 일도 수행 X -> 오래된 API 호출 결과를 무시하는 것임
  }, [userId]);
  ```